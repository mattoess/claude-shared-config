Run a quick dependency vulnerability audit.

## Process

### Step 1: Run npm audit

```bash
npm audit --json
```

### Step 2: Parse and Display Results

```
## Dependency Audit Results
Scan completed: {timestamp}

### Vulnerabilities Found

| Severity | Count |
|----------|-------|
| Critical | {n}   |
| High     | {n}   |
| Moderate | {n}   |
| Low      | {n}   |

### Critical Vulnerabilities
{For each critical vulnerability:}
- **Package**: {name}@{version}
- **CVE**: {cve_id}
- **Title**: {vulnerability_title}
- **Path**: {dependency_path}
- **Fix**: Upgrade to {fixed_version}

### High Vulnerabilities
{Similar format}

### Auto-Fixable
{count} vulnerabilities can be auto-fixed.

Run `npm audit fix` to apply safe fixes.
Run `npm audit fix --force` for breaking changes (review first!).
```

### Step 3: Offer Actions

After displaying results:

1. **Auto-fix safe updates?** (yes/no)
   - Runs `npm audit fix` if approved

2. **Show breaking changes?** (yes/no)
   - Lists updates that require `--force`

3. **Create ClickUp ticket for manual fixes?** (yes/no)
   - Creates ticket with vulnerability details

---

## Quick Commands

**Basic audit:**
```
/audit
```

**Audit with auto-fix:**
```
/audit --fix
```

**Production only:**
```
/audit --production
```

---

## Output Formats

**Summary** (default):
Shows counts and critical/high details

**Full**:
```
/audit --full
```
Shows all vulnerabilities including moderate/low

**JSON**:
```
/audit --json
```
Outputs raw npm audit JSON

---

## Common Fixes

### Safe Updates
```bash
# Apply safe updates
npm audit fix
```

### Breaking Changes
```bash
# Review first, then apply if safe
npm audit fix --force
```

### Specific Package
```bash
# Update specific package
npm update {package-name}
npm install {package-name}@latest
```

### Lock File Issues
```bash
# Regenerate lock file
rm package-lock.json
npm install
npm audit fix
```

---

## Integration

For comprehensive security scan including code analysis, use:
```
/security-scan
```

This audit command focuses specifically on dependency vulnerabilities (OWASP A06: Vulnerable and Outdated Components).
