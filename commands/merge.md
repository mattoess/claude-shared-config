Merge the current feature branch into a target branch.

## Usage

```
/merge
/merge to main
/merge to develop
```

---

## Process

### Step 1: Pre-Merge Validation

1. **Check Current Branch**
   ```bash
   git branch --show-current
   ```

   **Safety Check**: Prevent merging FROM main/develop
   ```
   Error: Currently on {main/develop} branch.

   This command merges feature branches INTO main/develop.
   You're already on the target branch.

   Did you mean to switch to a feature branch first?
   ```

2. **Check for Uncommitted Changes**
   ```bash
   git status
   ```

   **If uncommitted changes exist:**
   ```
   Uncommitted changes detected:
   {list of modified files}

   Please commit or stash changes first:
   - /commit (to commit changes)
   - git stash (to stash changes)

   Abort merge.
   ```

3. **Check for Unpushed Commits**
   ```bash
   git log origin/{current_branch}..HEAD
   ```

   **If unpushed commits exist:**
   ```
   You have {n} unpushed commit(s):
   {list commits}

   Push first? (yes/no/abort)
   ```

### Step 2: Determine Target Branch

**If user specified** (e.g., "/merge to main"):
- Use specified branch

**If not specified:**
```
Select target branch to merge into:
1. main
2. Other (specify)

Enter choice:
```

### Step 3: Confirm Merge Operation

```
About to merge:
  From: {current_branch}
  To:   {target_branch}

This will:
1. Switch to {target_branch}
2. Pull latest changes
3. Merge {current_branch} with --no-ff
4. Push to remote

Proceed? (yes/no)
```

### Step 4: Execute Merge

1. **Switch to Target Branch**
   ```bash
   git checkout {target_branch}
   ```

2. **Pull Latest Changes**
   ```bash
   git pull origin {target_branch}
   ```

3. **Perform Merge**
   ```bash
   git merge {feature_branch} --no-ff -m "Merge {feature_branch}: {description}"
   ```

### Step 5: Handle Merge Conflicts

**If conflicts occur:**
```
MERGE CONFLICTS DETECTED

Conflicted files:
{list of files with conflicts}

Steps to resolve:
1. Open each file and resolve conflicts
   - Look for <<<<<<< HEAD markers
   - Choose which changes to keep
   - Remove conflict markers

2. Stage resolved files:
   git add {file}

3. Complete the merge:
   git commit

Or abort merge:
   git merge --abort

I cannot auto-resolve conflicts. Please handle manually.
```

**Stop and wait for user to resolve.**

### Step 6: Push Changes

**If merge successful:**
```bash
git push origin {target_branch}
```

**Show Result:**
```
Merge completed successfully!

Merged: {feature_branch} -> {target_branch}
Commit: {merge_commit_hash}

{target_branch} has been updated on remote.
```

### Step 7: Update ClickUp (if applicable)

**If task ID found in branch name:**
Ask: "Update ClickUp task status to 'done'? (yes/no)"

If yes:
- Extract task ID from branch name
- Use `mcp__clickup__updateTask` to set status
- Add comment with merge details

### Step 8: Cleanup (Optional)

**Ask:**
```
Delete feature branch {feature_branch}? (yes/no)

This will:
- Delete local branch: git branch -d {feature_branch}
- Delete remote branch: git push origin --delete {feature_branch}
```

**If yes:**
```bash
git branch -d {feature_branch}
git push origin --delete {feature_branch}
```

---

## Merge Strategy

### Using --no-ff (No Fast-Forward)

Always use `--no-ff` flag to:
- Create explicit merge commit
- Preserve feature branch history
- Make it clear what was a feature branch

### Protected Branches

**Extra confirmation for main:**
```
WARNING: You're about to merge to MAIN branch.

This is typically done after:
- Code review (PR approved)
- All tests passing
- QA approval

Are you sure? (type 'yes' to confirm)
```

---

## Safeguards

### Pre-Merge Checks
- Not on target branch already
- No uncommitted changes
- All commits pushed
- Target branch is up to date

### During Merge
- Use --no-ff for explicit merge commit
- Stop if conflicts detected
- Never auto-resolve conflicts
- Clear instructions for manual resolution

### Post-Merge
- Confirm deletion before removing branches
- Show clear success/failure messages
- Optionally update ClickUp task status

### Never Do Without Confirmation
- Merge to main without warning
- Delete branches without asking
- Force push
- Auto-resolve conflicts

---

## Error Recovery

**If merge goes wrong:**

1. **Abort merge:**
   ```bash
   git merge --abort
   ```

2. **Return to previous state:**
   ```bash
   git checkout {feature_branch}
   ```

---

## Integration with ClickUp

After successful merge:
- Option to update task status to "done"
- Add merge commit reference to task comments
