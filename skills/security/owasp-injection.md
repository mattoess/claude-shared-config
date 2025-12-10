# OWASP A03:2021 - Injection

## Overview

Injection flaws occur when untrusted data is sent to an interpreter as part of a command or query. Hostile data can trick the interpreter into executing unintended commands or accessing data without authorization.

**Risk Level**: Critical

## Types of Injection

### 1. SQL Injection

**Vulnerable Code:**
```javascript
// String concatenation - VULNERABLE
const query = `SELECT * FROM users WHERE id = '${userId}'`
db.query(query)

// Template literals - VULNERABLE
db.query(`SELECT * FROM users WHERE email = '${email}'`)
```

**Secure Code:**
```javascript
// Parameterized queries - SECURE
db.query('SELECT * FROM users WHERE id = ?', [userId])

// Named parameters - SECURE
db.query('SELECT * FROM users WHERE email = :email', { email })

// ORM with proper escaping - SECURE
User.findOne({ where: { id: userId } })
```

**Detection Patterns:**
```regex
db\.(query|execute)\s*\(\s*`[^`]*\$\{
db\.(query|execute)\s*\(\s*['"][^'"]*\s*\+
SELECT.*FROM.*WHERE.*\$\{
INSERT.*INTO.*VALUES.*\$\{
```

### 2. NoSQL Injection

**Vulnerable Code:**
```javascript
// MongoDB - VULNERABLE
db.users.find({ username: req.body.username })

// With $where - EXTREMELY VULNERABLE
db.users.find({ $where: `this.username == '${username}'` })
```

**Secure Code:**
```javascript
// Type validation - SECURE
const username = String(req.body.username)
db.users.find({ username })

// Schema validation - SECURE
const schema = Joi.object({ username: Joi.string().alphanum() })
const { value } = schema.validate(req.body)
db.users.find({ username: value.username })

// Disable operators - SECURE
const sanitized = JSON.parse(JSON.stringify(req.body).replace(/\$/g, ''))
```

**Detection Patterns:**
```regex
\.find\(\s*\{[^}]*req\.(body|query|params)
\$where.*\$\{
\$where.*\+
```

### 3. Command Injection

**Vulnerable Code:**
```javascript
// exec with string - VULNERABLE
exec(`ls ${userInput}`)
exec('ping ' + hostname)

// shell: true - VULNERABLE
spawn('ls', ['-la'], { shell: true })
```

**Secure Code:**
```javascript
// execFile with array args - SECURE
execFile('ls', ['-la', sanitizedPath])

// spawn without shell - SECURE
spawn('ping', ['-c', '4', hostname], { shell: false })

// Whitelist approach - SECURE
const allowedCommands = ['list', 'status', 'info']
if (!allowedCommands.includes(command)) {
  throw new Error('Invalid command')
}
```

**Detection Patterns:**
```regex
exec\s*\(\s*`
exec\s*\(\s*['"][^'"]*\s*\+
spawn\s*\([^)]*shell:\s*true
child_process.*\$\{
```

### 4. XSS (Cross-Site Scripting)

**Vulnerable Code:**
```javascript
// innerHTML - VULNERABLE
element.innerHTML = userInput

// document.write - VULNERABLE
document.write(userInput)

// React dangerouslySetInnerHTML without sanitization - VULNERABLE
<div dangerouslySetInnerHTML={{ __html: userInput }} />
```

**Secure Code:**
```javascript
// textContent - SECURE
element.textContent = userInput

// DOMPurify sanitization - SECURE
import DOMPurify from 'dompurify'
element.innerHTML = DOMPurify.sanitize(userInput)

// React auto-escapes - SECURE
<div>{userInput}</div>

// Sanitize before dangerouslySetInnerHTML - SECURE
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userInput) }} />
```

**Detection Patterns:**
```regex
\.innerHTML\s*=
document\.write\s*\(
dangerouslySetInnerHTML.*\{.*__html:(?!.*sanitize)
v-html\s*=
\[innerHTML\]\s*=
```

### 5. LDAP Injection

**Vulnerable Code:**
```javascript
// String concatenation - VULNERABLE
const filter = `(uid=${username})`
ldap.search(base, { filter })
```

**Secure Code:**
```javascript
// Escape special characters - SECURE
const escapeLdap = (str) => str.replace(/[\\*()]/g, '\\$&')
const filter = `(uid=${escapeLdap(username)})`
```

### 6. Path Traversal

**Vulnerable Code:**
```javascript
// Direct path concatenation - VULNERABLE
const filePath = path.join(uploadDir, req.params.filename)
fs.readFile(filePath)
```

**Secure Code:**
```javascript
// Validate and normalize - SECURE
const filename = path.basename(req.params.filename)
const filePath = path.join(uploadDir, filename)

// Verify within allowed directory - SECURE
const resolved = path.resolve(uploadDir, filename)
if (!resolved.startsWith(path.resolve(uploadDir))) {
  throw new Error('Invalid path')
}
```

## Automated Checks

### ESLint Rules
```json
{
  "rules": {
    "security/detect-sql-injection": "error",
    "security/detect-non-literal-regexp": "warn",
    "security/detect-non-literal-fs-filename": "warn",
    "security/detect-child-process": "warn",
    "security/detect-eval-with-expression": "error"
  }
}
```

### Grep Patterns for Detection
```bash
# SQL Injection
grep -rn "query\s*(\s*\`" --include="*.ts" --include="*.js"
grep -rn "execute.*\+" --include="*.ts" --include="*.js"

# Command Injection
grep -rn "exec(\s*\`" --include="*.ts" --include="*.js"
grep -rn "spawn.*shell.*true" --include="*.ts" --include="*.js"

# XSS
grep -rn "innerHTML\s*=" --include="*.ts" --include="*.js"
grep -rn "dangerouslySetInnerHTML" --include="*.tsx" --include="*.jsx"
```

## Remediation Priority

1. **Critical**: SQL/NoSQL injection in auth or data access
2. **Critical**: Command injection with system access
3. **High**: XSS in user-facing applications
4. **High**: Path traversal with file access
5. **Medium**: LDAP injection in enterprise apps

## Testing

### Manual Testing
```bash
# SQL Injection test payloads
' OR '1'='1
'; DROP TABLE users; --
1 UNION SELECT * FROM users

# XSS test payloads
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>
javascript:alert('XSS')

# Command injection test payloads
; ls -la
| cat /etc/passwd
$(whoami)
```

## References

- [OWASP Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Injection_Prevention_Cheat_Sheet.html)
- [OWASP SQL Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [OWASP XSS Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
