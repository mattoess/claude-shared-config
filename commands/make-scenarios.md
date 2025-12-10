List, analyze, or trigger Make.com scenarios.

## Usage

```
/make-scenarios [action] [scenario-name-or-id]
```

## Actions

### List Scenarios (default)
```
/make-scenarios
/make-scenarios list
```

Shows all scenarios with:
- Name and ID
- Status (active/inactive)
- Last execution time
- Success rate

### Analyze Scenario
```
/make-scenarios analyze [name-or-id]
```

Deep dive into a specific scenario:
- Module breakdown
- Execution history
- Performance metrics
- Improvement suggestions

### Run Scenario
```
/make-scenarios run [name-or-id]
```

Trigger an on-demand scenario and report results.

### Debug Scenario
```
/make-scenarios debug [name-or-id]
```

Analyze recent failures and suggest fixes.

---

## Process

### For "list" action:

1. **Fetch all scenarios**
   ```
   Use list_scenarios to get all available scenarios
   ```

2. **Display summary table**
   ```markdown
   ## Make.com Scenarios

   | Name | Status | Last Run | Success Rate |
   |------|--------|----------|--------------|
   | Scenario 1 | Active | 2h ago | 98% |
   | Scenario 2 | Active | 1d ago | 100% |
   ```

3. **Offer actions**
   - "Which scenario would you like to analyze?"

### For "analyze" action:

1. **Get scenario details**
   ```
   Use get_scenario to fetch configuration
   ```

2. **Get execution history**
   ```
   Use list_executions to get recent runs
   ```

3. **Present analysis**
   ```markdown
   ## Scenario Analysis: [Name]

   ### Overview
   - **ID:** [id]
   - **Status:** [Active/Inactive]
   - **Scheduling:** [On-demand/Scheduled]
   - **Created:** [date]

   ### Modules ([count])
   1. **[Module Type]** - [App Name]
      - Purpose: [description]
   2. ...

   ### Execution Stats (Last 30 days)
   - Total Runs: [count]
   - Success Rate: [%]
   - Avg Duration: [time]
   - Operations/Run: [avg]

   ### Data Flow
   ```
   [Input] → [Process] → [Output]
   ```

   ### Recommendations
   1. [Improvement suggestion]
   2. [Improvement suggestion]
   ```

### For "run" action:

1. **Verify scenario exists and is on-demand**

2. **Confirm with user**
   ```
   Ready to run scenario "[Name]".
   This will use approximately X operations.
   Proceed? (yes/no)
   ```

3. **Execute and monitor**
   ```
   Use run_scenario to trigger
   Poll for completion
   ```

4. **Report results**
   ```markdown
   ## Execution Complete

   **Scenario:** [Name]
   **Status:** Success ✅ / Failed ❌
   **Duration:** [time]
   **Operations:** [count]

   ### Output
   [Scenario output data]
   ```

### For "debug" action:

1. **Get recent failed executions**
   ```
   Use list_executions filtered to failures
   ```

2. **Analyze failure patterns**
   ```
   Use get_execution_logs for each failure
   ```

3. **Present debug report**
   ```markdown
   ## Debug Report: [Scenario Name]

   ### Recent Failures ([count] in last 7 days)

   #### Failure 1 - [timestamp]
   - **Module:** [module name]
   - **Error:** [error message]
   - **Input:** [relevant input data]

   ### Root Cause Analysis
   [Explanation of why failures are occurring]

   ### Recommended Fixes
   1. [Specific fix with instructions]
   2. [Specific fix with instructions]

   ### Prevention
   [How to prevent future occurrences]
   ```

---

## Examples

**List all scenarios:**
```
/make-scenarios
```

**Analyze meeting prep scenario:**
```
/make-scenarios analyze "Meeting Prepper"
```

**Run a test:**
```
/make-scenarios run "Test Webhook Handler"
```

**Debug failures:**
```
/make-scenarios debug "Client Sync"
```

---

## Integration

This command works with:
- **Make MCP** - For scenario operations
- **ClickUp MCP** - Create tickets for issues found
- **Sentry MCP** - Correlate with application errors
