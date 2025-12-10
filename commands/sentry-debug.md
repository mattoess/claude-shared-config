Analyze and debug a Sentry error or issue.

## Usage

```
/sentry-debug [issue-id or search-term]
```

## Process

### Step 1: Find the Issue

**If issue ID provided:**
```
Use get_sentry_issue to fetch the issue details
```

**If search term provided:**
```
Use search_sentry_issues to find matching issues
Present top 5 results and ask user to select one
```

**If nothing provided:**
```
Use list_projects to show available projects
Ask: "Which project should I check for recent errors?"
Then fetch recent unresolved issues from that project
```

### Step 2: Gather Context

Once an issue is identified:

1. **Get stacktrace** - Full error trace with source maps
2. **Get breadcrumbs** - User actions before the error
3. **Get event context** - Browser, OS, user info
4. **Get tags** - Environment, release version, etc.

### Step 3: Analyze

Present the analysis:

```markdown
## Sentry Issue Analysis

**Issue:** [ISSUE-ID] [Error Title]
**Project:** [Project Name]
**Status:** [Unresolved/Resolved/Ignored]

### Impact
- **First Seen:** [Date]
- **Last Seen:** [Date]
- **Occurrences:** [Count]
- **Affected Users:** [Count]

### Error Details
**Type:** [Error Type]
**Message:** [Error Message]
**File:** [file:line]

### Stacktrace (Key Frames)
```
[Relevant stack frames]
```

### Breadcrumbs (Last 5 Actions)
1. [Action 1]
2. [Action 2]
...

### Environment
- **Browser:** [Browser info]
- **OS:** [OS info]
- **Release:** [Version]
```

### Step 4: Root Cause Analysis

**If Seer is available:**
```
Trigger Seer analysis for AI-powered root cause detection
Wait for analysis to complete
Present Seer's findings
```

**Manual analysis:**
```
Based on the stacktrace and context:
1. Identify the failing code
2. Explain why it fails
3. Check if related code exists in the codebase
```

### Step 5: Recommend Fix

```markdown
### Recommended Fix

**Location:** `[file:line]`

**Current Code:**
```javascript
[problematic code]
```

**Fixed Code:**
```javascript
[corrected code]
```

**Explanation:**
[Why this fix works]

**Testing:**
[How to verify the fix]
```

### Step 6: Offer Actions

After analysis, offer:

1. **View in Sentry** - Provide direct link to the issue
2. **Create ClickUp ticket** - Create a bug ticket with details
3. **Apply fix** - Edit the code to apply the recommended fix
4. **Mark resolved** - Mark the issue as resolved in Sentry
5. **Ignore issue** - Mark as ignored if it's expected behavior

---

## Examples

**Debug specific issue:**
```
/sentry-debug ISSUE-ABC123
```

**Search for errors:**
```
/sentry-debug TypeError undefined
```

**Check recent errors:**
```
/sentry-debug
```

---

## Integration

This command works best with:
- **Sentry MCP** - For fetching error data
- **ClickUp MCP** - For creating bug tickets
- **Security skill** - For identifying security-related errors
