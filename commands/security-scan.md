Run a comprehensive security scan on the codebase using OWASP Top 10 checks.

## Process

### Step 1: Dependency Audit

1. **Run npm audit**
   ```bash
   npm audit --json
   ```

2. **Report vulnerabilities by severity**
   ```
   Dependency Vulnerabilities:
   - Critical: {count}
   - High: {count}
   - Moderate: {count}
   - Low: {count}

   Details:
   {list critical and high vulnerabilities}
   ```

### Step 2: Code Security Scan

Search for common vulnerability patterns:

#### Injection Vulnerabilities (A03)
```bash
# SQL Injection
grep -rn "query.*\`\|execute.*\+" --include="*.ts" --include="*.js" src/

# Command Injection
grep -rn "exec\s*(\s*\`\|spawn.*shell.*true" --include="*.ts" --include="*.js" src/

# XSS
grep -rn "innerHTML\s*=\|dangerouslySetInnerHTML" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" src/
```

#### Authentication Issues (A07)
```bash
# Weak hashing
grep -rn "createHash.*md5\|sha1" --include="*.ts" --include="*.js" src/

# JWT without expiration
grep -rn "jwt\.sign" --include="*.ts" --include="*.js" src/

# Hardcoded secrets
grep -rn "password\s*=\s*['\"].*['\"]\|api_key\s*=\s*['\"]" --include="*.ts" --include="*.js" src/
```

#### Access Control Issues (A01)
```bash
# Missing authorization
grep -rn "findById\|findOne.*params" --include="*.ts" --include="*.js" src/

# Path traversal
grep -rn "path\.join.*params\|path\.join.*query" --include="*.ts" --include="*.js" src/
```

#### Security Misconfiguration (A05)
```bash
# Debug mode
grep -rn "DEBUG\s*=\s*true\|NODE_ENV.*development" --include="*.ts" --include="*.js" src/

# CORS wildcard
grep -rn "origin.*\*" --include="*.ts" --include="*.js" src/
```

### Step 3: Secrets Detection

Check for hardcoded secrets:
```bash
# API keys and tokens
grep -rn "sk-\|pk_\|api_key\|apiKey\|secret_key\|secretKey" --include="*.ts" --include="*.js" src/

# Private keys
grep -rn "BEGIN.*PRIVATE KEY\|-----BEGIN RSA" --include="*.ts" --include="*.js" --include="*.pem" src/

# AWS credentials
grep -rn "AKIA\|aws_access_key\|aws_secret" --include="*.ts" --include="*.js" src/
```

### Step 4: Report Findings

Format results as:

```
## Security Scan Results
Scan completed: {timestamp}
Files scanned: {count}

### Critical Issues (Fix Immediately)
1. **[A03] SQL Injection** in `src/api/users.ts:45`
   - Description: Raw SQL query with string interpolation
   - Fix: Use parameterized queries
   - Reference: skills/security/owasp-injection.md

### High Priority
2. **[A07] Hardcoded Secret** in `src/config.ts:12`
   - Description: API key hardcoded in source
   - Fix: Move to environment variable

### Medium Priority
3. **[A05] Debug Mode** in `src/app.ts:8`
   - Description: Debug mode enabled
   - Fix: Use environment-based configuration

### Low Priority / Informational
4. **[A06] Outdated Dependency** - lodash@4.17.15
   - Known CVE: CVE-2021-23337
   - Fix: npm update lodash

### Summary
- Critical: {count}
- High: {count}
- Medium: {count}
- Low: {count}

### Next Steps
1. Address critical issues immediately
2. Create tickets for high-priority items
3. Schedule medium/low for next sprint
```

---

## Scan Scope Options

**Full scan** (default):
```
/security-scan
```

**Specific directory:**
```
/security-scan src/api
```

**Specific check:**
```
/security-scan --check injection
/security-scan --check auth
/security-scan --check secrets
```

---

## Post-Scan Actions

After scan completes, offer:

1. **Create ClickUp tickets** for critical/high issues
2. **Auto-fix** simple issues (with confirmation)
3. **Generate detailed report** as markdown file
4. **Set up pre-commit hooks** for continuous scanning

---

## Integration

This command uses patterns from:
- `skills/security/skill.md` - Main security skill
- `skills/security/owasp-injection.md` - Injection patterns
- `skills/security/owasp-auth.md` - Auth patterns
- `skills/security/owasp-access-control.md` - Access control patterns
