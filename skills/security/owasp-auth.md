# OWASP A07:2021 - Identification and Authentication Failures

## Overview

Authentication failures occur when applications incorrectly confirm user identity, session management, or authentication mechanisms. This includes weak passwords, credential stuffing, session hijacking, and improper JWT handling.

**Risk Level**: Critical

## Common Vulnerabilities

### 1. Weak Password Policies

**Vulnerable Code:**
```javascript
// No password requirements - VULNERABLE
if (password.length > 0) {
  createUser(username, password)
}

// Simple length check only - VULNERABLE
if (password.length >= 6) {
  createUser(username, password)
}
```

**Secure Code:**
```javascript
// Strong password validation - SECURE
const passwordSchema = Joi.string()
  .min(12)
  .regex(/[A-Z]/)  // Uppercase
  .regex(/[a-z]/)  // Lowercase
  .regex(/[0-9]/)  // Number
  .regex(/[^A-Za-z0-9]/)  // Special char
  .required()

// Check against breached passwords - SECURE
const pwnedPasswords = require('hibp')
const isPwned = await pwnedPasswords.pwnedPassword(password)
if (isPwned) {
  throw new Error('Password found in data breach')
}
```

### 2. Insecure Password Storage

**Vulnerable Code:**
```javascript
// Plain text - EXTREMELY VULNERABLE
user.password = password

// MD5/SHA1 - VULNERABLE
user.password = crypto.createHash('md5').update(password).digest('hex')
user.password = crypto.createHash('sha1').update(password).digest('hex')

// SHA256 without salt - VULNERABLE
user.password = crypto.createHash('sha256').update(password).digest('hex')
```

**Secure Code:**
```javascript
// bcrypt - SECURE
const bcrypt = require('bcrypt')
const SALT_ROUNDS = 12

// Hash password
user.password = await bcrypt.hash(password, SALT_ROUNDS)

// Verify password
const isValid = await bcrypt.compare(password, user.password)

// Argon2 - ALSO SECURE (preferred for new projects)
const argon2 = require('argon2')
user.password = await argon2.hash(password)
const isValid = await argon2.verify(user.password, password)
```

### 3. Missing Rate Limiting

**Vulnerable Code:**
```javascript
// No rate limiting - VULNERABLE
app.post('/login', async (req, res) => {
  const { username, password } = req.body
  const user = await User.findOne({ username })
  // ... authenticate
})
```

**Secure Code:**
```javascript
// Rate limiting - SECURE
const rateLimit = require('express-rate-limit')

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts per window
  message: 'Too many login attempts',
  standardHeaders: true,
  legacyHeaders: false,
})

app.post('/login', loginLimiter, async (req, res) => {
  // ... authenticate
})

// Account lockout - SECURE
const MAX_ATTEMPTS = 5
const LOCKOUT_DURATION = 30 * 60 * 1000 // 30 minutes

if (user.failedAttempts >= MAX_ATTEMPTS) {
  const lockoutEnd = user.lockoutUntil
  if (lockoutEnd && lockoutEnd > Date.now()) {
    throw new Error('Account locked. Try again later.')
  }
}
```

### 4. Insecure JWT Implementation

**Vulnerable Code:**
```javascript
// No expiration - VULNERABLE
const token = jwt.sign({ userId: user.id }, secret)

// Algorithm none attack - VULNERABLE
const decoded = jwt.verify(token, secret)

// Weak secret - VULNERABLE
const secret = 'password123'

// Sensitive data in payload - VULNERABLE
const token = jwt.sign({
  userId: user.id,
  password: user.password,
  ssn: user.ssn
}, secret)
```

**Secure Code:**
```javascript
// Proper JWT implementation - SECURE
const JWT_SECRET = process.env.JWT_SECRET // Strong, from env
const ACCESS_TOKEN_EXPIRY = '15m'
const REFRESH_TOKEN_EXPIRY = '7d'

// Sign with expiration and algorithm
const accessToken = jwt.sign(
  { userId: user.id, role: user.role },
  JWT_SECRET,
  {
    expiresIn: ACCESS_TOKEN_EXPIRY,
    algorithm: 'HS256'
  }
)

// Verify with algorithm restriction
const decoded = jwt.verify(token, JWT_SECRET, {
  algorithms: ['HS256'] // Prevent algorithm confusion
})

// Refresh token rotation
const refreshToken = crypto.randomBytes(64).toString('hex')
await RefreshToken.create({
  token: refreshToken,
  userId: user.id,
  expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
})
```

### 5. Session Management Issues

**Vulnerable Code:**
```javascript
// Session ID in URL - VULNERABLE
app.get('/dashboard?sessionId=abc123', ...)

// No session regeneration - VULNERABLE
req.session.userId = user.id
// Session ID stays the same after login

// No secure flags - VULNERABLE
app.use(session({
  secret: 'secret',
  cookie: {}
}))
```

**Secure Code:**
```javascript
// Secure session configuration - SECURE
app.use(session({
  secret: process.env.SESSION_SECRET,
  name: '__Host-sessionId', // Cookie prefix
  cookie: {
    secure: true,        // HTTPS only
    httpOnly: true,      // No JS access
    sameSite: 'strict',  // CSRF protection
    maxAge: 3600000      // 1 hour
  },
  resave: false,
  saveUninitialized: false,
  store: new RedisStore({ client: redisClient })
}))

// Regenerate session on login - SECURE
app.post('/login', async (req, res) => {
  // ... verify credentials
  req.session.regenerate((err) => {
    req.session.userId = user.id
    res.json({ success: true })
  })
})

// Destroy session on logout - SECURE
app.post('/logout', (req, res) => {
  req.session.destroy((err) => {
    res.clearCookie('__Host-sessionId')
    res.json({ success: true })
  })
})
```

### 6. OAuth/OIDC Vulnerabilities

**Vulnerable Code:**
```javascript
// Missing state parameter - VULNERABLE
const authUrl = `${provider}/auth?client_id=${clientId}&redirect_uri=${redirectUri}`

// Not validating redirect URI - VULNERABLE
app.get('/callback', (req, res) => {
  const { code, redirect } = req.query
  res.redirect(redirect) // Open redirect!
})
```

**Secure Code:**
```javascript
// State parameter for CSRF - SECURE
const state = crypto.randomBytes(32).toString('hex')
req.session.oauthState = state

const authUrl = new URL(`${provider}/auth`)
authUrl.searchParams.set('client_id', clientId)
authUrl.searchParams.set('redirect_uri', redirectUri)
authUrl.searchParams.set('state', state)
authUrl.searchParams.set('response_type', 'code')

// Validate state on callback - SECURE
app.get('/callback', (req, res) => {
  const { code, state } = req.query
  if (state !== req.session.oauthState) {
    throw new Error('Invalid state parameter')
  }
  delete req.session.oauthState
  // ... exchange code for token
})

// Whitelist redirect URIs - SECURE
const ALLOWED_REDIRECTS = ['/dashboard', '/profile']
if (!ALLOWED_REDIRECTS.includes(redirect)) {
  throw new Error('Invalid redirect')
}
```

## Detection Patterns

```bash
# Weak hashing
grep -rn "createHash.*md5\|sha1" --include="*.ts" --include="*.js"

# JWT without expiration
grep -rn "jwt\.sign.*[^{]*\)" --include="*.ts" --include="*.js"

# Missing rate limiting
grep -rn "app\.\(post\|put\).*login\|auth\|signin" --include="*.ts" --include="*.js"

# Session without secure flags
grep -rn "session\s*(\s*{" --include="*.ts" --include="*.js"
```

## Automated Checks

### ESLint Rules
```json
{
  "rules": {
    "security/detect-possible-timing-attacks": "error"
  }
}
```

## Remediation Checklist

- [ ] Passwords hashed with bcrypt/argon2 (cost factor >= 12)
- [ ] Rate limiting on auth endpoints
- [ ] Account lockout after failed attempts
- [ ] JWT with expiration and algorithm restriction
- [ ] Session regeneration on authentication
- [ ] Secure cookie flags (Secure, HttpOnly, SameSite)
- [ ] OAuth state parameter validation
- [ ] MFA available for sensitive accounts

## References

- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [OWASP Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
