# OWASP A01:2021 - Broken Access Control

## Overview

Broken Access Control is the #1 vulnerability in OWASP Top 10 2021. It occurs when users can access resources or perform actions outside their intended permissions. This includes IDOR, privilege escalation, and missing authorization checks.

**Risk Level**: Critical

## Common Vulnerabilities

### 1. Insecure Direct Object References (IDOR)

**Vulnerable Code:**
```javascript
// No ownership check - VULNERABLE
app.get('/api/documents/:id', async (req, res) => {
  const doc = await Document.findById(req.params.id)
  res.json(doc)
})

// User can access any invoice - VULNERABLE
app.get('/api/invoices/:invoiceId', async (req, res) => {
  const invoice = await Invoice.findById(req.params.invoiceId)
  res.json(invoice)
})
```

**Secure Code:**
```javascript
// Ownership validation - SECURE
app.get('/api/documents/:id', async (req, res) => {
  const doc = await Document.findOne({
    _id: req.params.id,
    ownerId: req.user.id  // Ensure ownership
  })

  if (!doc) {
    return res.status(404).json({ error: 'Not found' })
  }

  res.json(doc)
})

// Role-based access - SECURE
app.get('/api/invoices/:invoiceId', async (req, res) => {
  const invoice = await Invoice.findById(req.params.invoiceId)

  if (!invoice) {
    return res.status(404).json({ error: 'Not found' })
  }

  // Check if user owns or has permission
  const hasAccess =
    invoice.userId === req.user.id ||
    invoice.organizationId === req.user.organizationId ||
    req.user.role === 'admin'

  if (!hasAccess) {
    return res.status(403).json({ error: 'Forbidden' })
  }

  res.json(invoice)
})
```

### 2. Missing Function Level Access Control

**Vulnerable Code:**
```javascript
// No auth check on admin route - VULNERABLE
app.delete('/api/users/:id', async (req, res) => {
  await User.deleteOne({ _id: req.params.id })
  res.json({ success: true })
})

// Hidden endpoint without protection - VULNERABLE
app.post('/api/admin/reset-all', async (req, res) => {
  await resetDatabase()
  res.json({ success: true })
})
```

**Secure Code:**
```javascript
// Middleware for role checking - SECURE
const requireRole = (...roles) => (req, res, next) => {
  if (!req.user) {
    return res.status(401).json({ error: 'Unauthorized' })
  }

  if (!roles.includes(req.user.role)) {
    return res.status(403).json({ error: 'Forbidden' })
  }

  next()
}

// Protected admin routes - SECURE
app.delete('/api/users/:id',
  requireAuth,
  requireRole('admin'),
  async (req, res) => {
    await User.deleteOne({ _id: req.params.id })
    res.json({ success: true })
  }
)

// All admin routes protected - SECURE
app.use('/api/admin', requireAuth, requireRole('admin'))
```

### 3. Path Traversal

**Vulnerable Code:**
```javascript
// Direct path concatenation - VULNERABLE
app.get('/files/:filename', (req, res) => {
  const filePath = path.join(UPLOAD_DIR, req.params.filename)
  res.sendFile(filePath)
})
// Attack: /files/../../../etc/passwd
```

**Secure Code:**
```javascript
// Path validation - SECURE
app.get('/files/:filename', (req, res) => {
  // Remove directory components
  const filename = path.basename(req.params.filename)
  const filePath = path.join(UPLOAD_DIR, filename)

  // Verify resolved path is within allowed directory
  const resolvedPath = path.resolve(filePath)
  const allowedDir = path.resolve(UPLOAD_DIR)

  if (!resolvedPath.startsWith(allowedDir + path.sep)) {
    return res.status(403).json({ error: 'Invalid path' })
  }

  res.sendFile(resolvedPath)
})
```

### 4. Privilege Escalation

**Vulnerable Code:**
```javascript
// User can set their own role - VULNERABLE
app.put('/api/profile', async (req, res) => {
  const { name, email, role } = req.body
  await User.updateOne(
    { _id: req.user.id },
    { name, email, role }
  )
})

// Mass assignment vulnerability - VULNERABLE
app.post('/api/users', async (req, res) => {
  const user = await User.create(req.body)
  res.json(user)
})
```

**Secure Code:**
```javascript
// Whitelist allowed fields - SECURE
app.put('/api/profile', async (req, res) => {
  const { name, email } = req.body // Only allow specific fields
  await User.updateOne(
    { _id: req.user.id },
    { name, email }  // Role cannot be changed here
  )
})

// Explicit field selection - SECURE
app.post('/api/users', async (req, res) => {
  const { name, email, password } = req.body
  const user = await User.create({
    name,
    email,
    password,
    role: 'user'  // Always default role
  })
  res.json(user)
})

// DTO pattern - SECURE
const createUserDto = (body) => ({
  name: body.name,
  email: body.email,
  password: body.password,
  role: 'user',  // Hardcoded default
  createdAt: new Date()
})
```

### 5. CORS Misconfiguration

**Vulnerable Code:**
```javascript
// Allow all origins - VULNERABLE
app.use(cors({ origin: '*' }))

// Reflect origin header - VULNERABLE
app.use(cors({
  origin: (origin, callback) => callback(null, origin),
  credentials: true
}))
```

**Secure Code:**
```javascript
// Whitelist specific origins - SECURE
const allowedOrigins = [
  'https://app.example.com',
  'https://admin.example.com'
]

app.use(cors({
  origin: (origin, callback) => {
    // Allow requests with no origin (mobile apps, curl)
    if (!origin) return callback(null, true)

    if (allowedOrigins.includes(origin)) {
      callback(null, true)
    } else {
      callback(new Error('Not allowed by CORS'))
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization']
}))
```

### 6. Missing Authorization on API Endpoints

**Vulnerable Code:**
```javascript
// Some routes protected, others not - VULNERABLE
app.get('/api/public/posts', getPosts)
app.get('/api/users', getUsers)  // Should require auth!
app.put('/api/settings', updateSettings)  // Should require auth!
```

**Secure Code:**
```javascript
// Default deny - all routes require auth - SECURE
app.use('/api', requireAuth)

// Explicitly allow public routes
app.get('/api/public/posts', getPosts)
app.post('/api/auth/login', login)
app.post('/api/auth/register', register)

// Protected by default
app.get('/api/users', requireRole('admin'), getUsers)
app.put('/api/settings', updateSettings)
```

## Access Control Patterns

### Role-Based Access Control (RBAC)
```javascript
const permissions = {
  admin: ['read', 'write', 'delete', 'manage_users'],
  editor: ['read', 'write'],
  viewer: ['read']
}

const hasPermission = (role, permission) => {
  return permissions[role]?.includes(permission) ?? false
}

const requirePermission = (permission) => (req, res, next) => {
  if (!hasPermission(req.user.role, permission)) {
    return res.status(403).json({ error: 'Forbidden' })
  }
  next()
}
```

### Attribute-Based Access Control (ABAC)
```javascript
const canAccess = (user, resource, action) => {
  // Owner can do anything
  if (resource.ownerId === user.id) return true

  // Organization members can read
  if (action === 'read' && resource.orgId === user.orgId) return true

  // Admins can do anything
  if (user.role === 'admin') return true

  return false
}
```

## Detection Patterns

```bash
# Missing auth middleware
grep -rn "app\.\(get\|post\|put\|delete\).*async.*req.*res" --include="*.ts" --include="*.js" | grep -v "requireAuth\|isAuthenticated"

# Direct ID access without ownership check
grep -rn "findById\|findOne.*params\." --include="*.ts" --include="*.js"

# CORS wildcard
grep -rn "origin.*\*\|origin.*true" --include="*.ts" --include="*.js"

# Path concatenation
grep -rn "path\.join.*params\.\|path\.join.*query\." --include="*.ts" --include="*.js"
```

## Remediation Checklist

- [ ] All endpoints have authentication checks
- [ ] All endpoints have authorization checks
- [ ] IDOR prevention with ownership validation
- [ ] Role-based or attribute-based access control
- [ ] Path traversal prevention
- [ ] Mass assignment protection
- [ ] CORS properly configured
- [ ] Default deny policy for new routes
- [ ] Audit logging for access control failures

## Testing

```bash
# IDOR testing
# Change IDs in URLs to access other users' resources
GET /api/documents/123  # Your document
GET /api/documents/456  # Someone else's document

# Privilege escalation testing
# Try adding role to request body
PUT /api/profile
{ "name": "Test", "role": "admin" }

# Path traversal testing
GET /files/../../../../etc/passwd
GET /files/..%2F..%2F..%2Fetc/passwd
```

## References

- [OWASP Access Control Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Access_Control_Cheat_Sheet.html)
- [OWASP IDOR Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html)
- [OWASP Authorization Testing](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/)
