Create a git commit following Conventional Commits specification with ClickUp task reference.

## Process

### Step 1: Review Changes

1. **Check Git Status**
   ```bash
   git status
   ```
   Show modified, added, deleted files.

2. **Show Diff**
   ```bash
   git diff
   ```
   Display actual changes (limit output if too large).

3. **Ask user**: "Review changes above. Proceed with commit? (yes/no/show more)"

### Step 2: Stage Changes

**Ask**: "Stage all changes, or specific files?"

**If all:**
```bash
git add .
```

**If specific:**
Ask: "Which files to stage?"
```bash
git add {specified_files}
```

**Verify staged changes:**
```bash
git diff --staged
```

### Step 3: Generate Commit Message

1. **Analyze Changes**
   - Determine commit type (feat, fix, docs, etc.)
   - Identify scope (e.g., auth, api, ui)
   - Summarize changes concisely

2. **Extract Ticket ID**
   - Get current branch name: `git branch --show-current`
   - Parse ticket ID from branch name pattern: `CU-{task_id}_{description}_{developer_name}`

3. **Construct Message**
   ```
   {type}({scope}): {description} [{ticket-id}]

   {bullet points of main changes}
   ```

4. **Show Proposed Message**
   ```
   Proposed commit message:

   feat(salesforce): Add OAuth2 login flow [CU-86b7bbrtq]

   - Implemented OAuth2 provider integration
   - Added JWT token management
   - Added user session persistence

   Approve? (yes/no/edit)
   ```

### Step 4: Create Commit

**If approved:**
```bash
git commit -m "$(cat <<'EOF'
{approved_message}
EOF
)"
```

**If user wants to edit:**
- Ask for changes
- Update message
- Show again for approval

**Show Result:**
```
Commit created: {commit_hash}
  {commit_message_first_line}

Next steps:
- Continue working
- git push to push to remote
- /merge to merge branch
```

---

## Commit Message Guidelines

### Types
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Formatting, whitespace
- `refactor`: Code refactoring
- `test`: Adding/updating tests
- `chore`: Maintenance
- `perf`: Performance improvement

### Format Rules
- First line max 72 characters
- Use imperative mood ("Add feature" not "Added feature")
- Include ticket ID in brackets: `[CU-86b7bbrtq]`
- Body explains what and why (not how)

### Examples

**Feature:**
```
feat(salesforce): Add OAuth2 login flow [CU-86b7bbrtq]

Implemented OAuth2 provider integration with JWT tokens.
Users can now log in with Salesforce credentials.
```

**Bug Fix:**
```
fix(api): Resolve timeout in org switch [CU-abc123xyz]

Increased timeout from 5s to 30s for org context switching.
Added retry logic with exponential backoff.
```

---

## Safeguards

### Before Committing, Check:

- No sensitive data (API keys, passwords, tokens)
- No debug code left (console.logs, debuggers)
- No commented-out code blocks (unless intentional)
- No temporary files (.DS_Store, .log, etc.)

**If found, warn user:**
```
Warning: Found potential issues:
- API key in config.ts line 42
- console.log in auth.ts line 156

Remove these before committing? (yes/no)
```

### Files to Never Commit

Check if staging:
- `.env` files
- `node_modules/`
- Build artifacts (`dist/`, `build/`)
- IDE files (`.vscode/`, `.idea/`)
- OS files (`.DS_Store`, `Thumbs.db`)
- Log files (`*.log`)
- Credential files

---

## Error Handling

**No changes to commit:**
```
No changes staged for commit.

Options:
- Modify files and try again
- Use git add to stage changes
- Check git status
```

**No ticket ID found in branch name:**
```
Warning: Could not extract ticket ID from branch name.

Current branch: {branch_name}

Expected format: CU-{task_id}_{description}_{developer_name}

Commit without ticket reference? (yes/no)
```

---

## Integration with ClickUp

- Commits with proper branch naming auto-link to ClickUp
- ClickUp task shows all commits from the branch
- Commit messages appear in ClickUp task activity
