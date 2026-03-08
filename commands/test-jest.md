Analyze recent changes and generate targeted Jest tests, then run them and report results.

## Usage

```
/test-jest
/test-jest api/airtable/sessions
/test-jest --all
```

---

## Process

### Step 1: Identify What Changed

1. **Find the base branch and changed files**
   ```bash
   git merge-base HEAD origin/main
   git diff --name-only $(git merge-base HEAD origin/main)..HEAD
   ```

2. **Filter to testable files** — only `.js`, `.ts`, `.tsx` files in `src/` and `api/`
   - Exclude test files themselves, config files, `.md`, `.css`, `.json`
   - Exclude `node_modules/`, `dist/`, `build/`

3. **If a specific path was provided** (e.g., `/test-jest api/airtable/sessions`):
   - Scope to only that directory
   - Still check what changed within it

4. **Show summary:**
   ```
   Changed files since main:
   - api/salesforce/canvas.js
   - src/components/ThemesReview.tsx
   - api/hubspot/oauth-callback.js

   Proceed with test generation? (yes/no/select specific files)
   ```

### Step 2: Check Existing Test Coverage

For each changed file, check if tests already exist:

1. **Find existing test files**
   - API routes: look for `__tests__/route.test.js` in same directory
   - Frontend components: look for `*.test.ts`, `*.test.tsx`, `*.spec.ts` alongside or in `__tests__/`
   - Services: look for `*.test.ts` in same directory or `__tests__/`

2. **Read existing tests** to understand:
   - What's already covered (don't duplicate)
   - Testing patterns and conventions used
   - Mock setup and utilities

3. **Report coverage status:**
   ```
   Test coverage status:
   ✅ api/airtable/sessions/update-session — has tests (route.test.js)
   ⚠️  api/salesforce/canvas.js — has tests but may need updates
   ❌ src/components/ThemesReview.tsx — no tests found

   Generate tests for uncovered/outdated files? (yes/no/select)
   ```

### Step 3: Generate Tests

Follow these conventions strictly:

**API route tests** (`api/**/__tests__/route.test.js`):
- Use Jest with `describe`/`it` blocks
- Mock Airtable, Clerk auth, and external services
- Test: valid requests, auth failures, validation errors, edge cases
- Follow the existing pattern in `api/__tests__/setup.js`
- Use test env vars (e.g., `process.env.AIRTABLE_API_KEY = 'test-api-key'`)
- Test both success and error paths

**Frontend component tests** (`src/**/*.test.tsx`):
- Use `@testing-library/react` with `render`, `screen`, `fireEvent`
- Mock Zustand stores and service calls
- Test: rendering, user interactions, conditional displays, error states
- Follow patterns in existing `src/tests/` directory

**Service tests** (`src/services/**/*.test.ts`):
- Mock fetch/API calls
- Test: successful responses, error handling, data transformation
- Verify auth token is passed correctly

**For each test file, show the user:**
```
Proposed test file: api/salesforce/__tests__/canvas.test.js

Tests:
1. should return 401 without valid auth
2. should return 400 without signed request
3. should verify and decode valid signed request
4. should handle invalid consumer secret gracefully
5. should apply rate limiting

Write this file? (yes/no/edit)
```

### Step 4: Run Tests

1. **Run only the new/changed tests first** (fast feedback):
   ```bash
   npx jest --testPathPattern="{pattern}" --verbose
   ```

2. **If new tests pass, run the full suite:**
   ```bash
   npm test
   ```

3. **If `--all` flag was provided, run everything:**
   ```bash
   npm test
   npm run test:backend
   ```

### Step 5: Report Results

```
## Test Results

### New Tests Created
- api/salesforce/__tests__/canvas.test.js (5 tests)
- src/components/__tests__/ThemesReview.test.tsx (8 tests)

### Results
✅ New tests: 13/13 passing
✅ Full suite: 47/47 passing

### Coverage Delta
- api/salesforce/canvas.js: 0% → 85%
- src/components/ThemesReview.tsx: 0% → 72%

### Failed Tests (if any)
❌ ThemesReview.test.tsx > should block save when themes exceed limit
   Expected: warning message displayed
   Actual: save proceeded without validation

   Suggested fix: [explanation]
   Auto-fix? (yes/no)
```

---

## Test Conventions

### File Location
- API routes: `api/{domain}/{action}/__tests__/route.test.js`
- Frontend: `src/components/__tests__/{Component}.test.tsx` or alongside
- Services: `src/services/__tests__/{service}.test.ts`

### Naming
- Describe blocks match function/component name
- Test names start with "should" and describe the expected behavior
- Group by feature area: auth, validation, happy path, error handling

### Mocking
- Airtable: mock the `base().table().select/create/update` chain
- Clerk: mock `requireClerkAuth` to inject `req.auth = { userId, orgId }`
- External APIs: mock `fetch` or specific SDK methods
- Never make real API calls in tests

---

## Error Handling

**No changes found:**
```
No testable files changed since main.

Options:
- /test-jest --all (run full test suite)
- /test-jest src/components (test specific directory)
```

**Test generation fails:**
- Show the error and the file that caused it
- Skip that file and continue with others
- Ask user if they want to manually write the test

**Tests fail after generation:**
- Show failure details with line numbers
- Offer to auto-fix if the issue is in the test (not the source)
- If source code has a bug, report it clearly — don't modify source code
