Full quality gate pipeline: build, test, lint, security scan, then commit, push, and create PR.

## Usage

```
/commit-to-pr
/commit-to-pr to main
```

---

## Process

### Phase 1: Build & Lint (Fast Fail)

Run build and lint in sequence — fail fast before slower checks.

1. **Build check**
   ```bash
   npm run build
   ```

   **If build fails:**
   ```
   ❌ BUILD FAILED

   {error output}

   Fix build errors before proceeding. Aborting.
   ```
   Stop here. Do not continue.

2. **Lint check**
   ```bash
   npm run lint
   ```

   **If lint fails:**
   ```
   ⚠️  LINT ISSUES FOUND

   {error output}

   Auto-fix lint issues? (yes/no/abort)
   ```
   If yes, run `npm run lint -- --fix`, then re-check.
   If lint errors remain after fix, show them and ask user to resolve.

### Phase 2: Tests

1. **Run Jest tests**
   ```bash
   npm test
   ```

   **If tests fail:**
   ```
   ❌ TESTS FAILED

   {failure summary}

   Fix failing tests before proceeding. Aborting.
   ```
   Stop here. Do not continue.

2. **Show results:**
   ```
   ✅ Tests: {n} passing, 0 failing
   ```

### Phase 3: Security Checks

Run dependency audit and code security scan.

1. **Dependency audit**
   ```bash
   npm audit --json
   ```

2. **Code security scan** — check for OWASP Top 10 patterns:
   - Injection vulnerabilities (eval, innerHTML, dangerouslySetInnerHTML, unsanitized queries)
   - Hardcoded secrets (API keys, passwords, tokens in source — not test files)
   - Partial secret logging (console.log with key/secret/token substrings)
   - Missing auth middleware on API routes
   - Path traversal (user input in path.join/resolve)
   - CORS wildcard origins
   - Debug flags left enabled

3. **Evaluate results with blocking thresholds:**

   **BLOCK (stop and require fix):**
   - Any critical or high severity dependency vulnerability **introduced by this branch** (not pre-existing)
   - Any code security finding rated critical or high (hardcoded secrets, injection, missing auth)

   **WARN (show but continue):**
   - Pre-existing critical/high dependency vulnerabilities (already tracked)
   - Moderate and low dependency vulnerabilities
   - Moderate/low code findings (verbose logging, debug flags)

   **To determine if a vulnerability is new vs pre-existing:**
   ```bash
   # Check if vulnerability existed on main
   git stash
   git checkout main
   npm audit --json > /tmp/main-audit.json
   git checkout -
   git stash pop
   ```
   Compare current audit against main's baseline. Only block on net-new issues.

4. **Show security summary:**
   ```
   ## Security Check Results

   ### Dependency Audit
   ✅ No new vulnerabilities introduced
   ⚠️  34 pre-existing vulnerabilities (7 high, 18 moderate, 9 low)
      Tracked in: https://app.clickup.com/t/86b8tc3nh

   ### Code Security Scan
   ✅ No critical/high issues found
   ⚠️  1 moderate: verbose logging in api/hubspot/token-manager.js:55

   Proceed? (yes/no)
   ```

   **If blocked:**
   ```
   🚫 SECURITY GATE FAILED

   New critical/high issues found:
   1. [A07] Hardcoded API key in src/config.ts:42
   2. [HIGH] New vulnerable dependency: lodash@4.17.15

   These must be resolved before creating a PR. Aborting.
   ```

### Phase 4: Commit

Follow the `/commit` process:

1. **Review changes** — `git status` and `git diff`
2. **Stage files by name** — never `git add .`
3. **Generate detailed commit message** with:
   - Conventional commit format: `{type}({scope}): {description} [{ticket-id}]`
   - Detailed body explaining what changed and why
   - ClickUp task ID extracted from branch name
   - `Co-Authored-By: Claude <noreply@anthropic.com>`
4. **Show proposed message** for approval
5. **Create commit**

### Phase 5: Push & Create PR

1. **Push to remote**
   ```bash
   git push -u origin {current_branch}
   ```

2. **Determine target branch**
   - Default: `main`
   - If user specified (e.g., `/commit-to-pr to develop`), use that

3. **Generate PR description** from ALL commits on the branch (not just the latest):
   ```bash
   git log --oneline $(git merge-base HEAD origin/main)..HEAD
   ```

4. **Create PR:**
   ```bash
   gh pr create --title "{title}" --body "$(cat <<'EOF'
   ## Summary
   {2-4 bullet points covering all commits on the branch}

   ## Quality Gates
   - ✅ Build: passing
   - ✅ Lint: passing
   - ✅ Tests: {n} passing
   - ✅ Security: no new critical/high issues
   - ⚠️  Known: {n} pre-existing dependency vulnerabilities (tracked)

   ## Test plan
   {bulleted checklist of things to verify}

   🤖 Generated with [Claude Code](https://claude.com/claude-code)
   EOF
   )"
   ```

5. **Show result:**
   ```
   ✅ PR created successfully!

   PR: {url}
   Branch: {branch} → {target}
   Commits: {n}

   Quality gates passed:
   - Build ✅
   - Lint ✅
   - Tests ✅ ({n} passing)
   - Security ✅ (no new issues)

   Next steps:
   - Review PR at {url}
   - Request reviewers if needed
   - /merge after approval
   ```

### Phase 6: Update ClickUp (Optional)

**If task ID found in branch name:**
```
Update ClickUp task status to 'in review'? (yes/no)
```

If yes:
- Update task status via `mcp__clickup__updateTask`
- Add comment with PR link

---

## Pipeline Summary

```
Phase 1: Build & Lint     → FAIL = abort
Phase 2: Tests             → FAIL = abort
Phase 3: Security          → NEW critical/high = abort
                           → Pre-existing/moderate/low = warn & continue
Phase 4: Commit            → Stage, message, commit
Phase 5: Push & PR         → Push, create PR with quality gate summary
Phase 6: ClickUp           → Optional status update
```

---

## Error Recovery

**Build fails:**
- Show errors, abort. User must fix and re-run.

**Tests fail:**
- Show failures with details, abort. User must fix and re-run.
- Suggest: "Run `/test-jest` to investigate and fix failing tests"

**Security blocks:**
- Show specific issues with file:line references
- Suggest fixes where possible
- Abort. User must fix and re-run.

**Push fails:**
- If remote has new commits: suggest `git pull --rebase`
- If branch protection: show the protection rule and suggest alternatives

**PR creation fails:**
- If PR already exists: show existing PR URL, offer to update it
- If no upstream: push with `-u` flag first
