You are helping work on a ClickUp ticket. Follow these steps IN ORDER:

## Step 1: Fetch Ticket from ClickUp

1. **Get the ticket**
   - User will provide ticket ID (e.g., "86b7bbrtq" or "CU-86b7bbrtq")
   - If they provide "CU-" prefix, strip it for the search
   - Use `mcp__clickup__getTaskById`
   - Handle errors gracefully

2. **Display Ticket Information**
   - Show formatted ticket details
   - Highlight acceptance criteria
   - Note any blockers or dependencies
   - Include the ClickUp URL

## Step 2: Provide Branch Creation Command

1. **Generate Branch Name**
   - Extract task ID from ClickUp
   - Read developer name from `clickup.json`
   - Use pattern: `CU-{task_id}_{short-description}_{developer_name}`
   - Keep description concise (2-4 words, kebab-case)

2. **Show Command**
   ```
   Suggested Branch:
   git checkout -b CU-{task_id}_{description}_{developer_name}
   ```

3. **Explain**
   - "Run this command when ready to begin work"
   - "This branch naming convention auto-links commits to ClickUp"

## Step 3: Analyze Codebase (if needed)

1. **Search for Related Code**
   - Use Grep/Glob to find related files
   - Identify areas that need changes
   - Look for similar patterns in codebase

2. **Suggest Implementation Approach**
   - Outline files to modify
   - Suggest architectural approach
   - Identify potential challenges

## Step 4: Create TODO List

1. **Use TodoWrite tool** to create task list
   - Break ticket into implementation steps
   - Include testing requirements
   - Add documentation tasks if needed

## Step 5: Offer to Update Task Status

Ask: "Ready to start work? I can update the task status to 'in progress'. (yes/no)"

If yes:
- Use `mcp__clickup__updateTask` to set status
- Confirm update

If no:
- Acknowledge
- Let user know to say "start work" when ready

---

## Prerequisites
- ClickUp MCP server connected
- `clickup.json` in project root
- Git repository initialized
- Clean working directory (check with `git status`)

## Error Handling

**If ticket not found:**
- Display error message
- Ask user to verify ticket ID
- Suggest searching by task name instead

**If branch already exists:**
- Show existing branch info
- Ask if user wants to switch to it
- Or suggest a different branch name

**If uncommitted changes:**
- Show git status
- Ask user to commit or stash first
- Do not proceed until clean

---

## Example Usage

```
User: /ticket 86b7bbrtq

Claude:
1. Fetches ClickUp ticket 86b7bbrtq
2. Displays ticket details
3. Shows: git checkout -b CU-86b7bbrtq_salesforce-integration_Developer-Name
4. Analyzes relevant codebase files
5. Creates TODO list for implementation
6. Asks if ready to update status
```

---

## Safeguards

- Always verify git status is clean before suggesting branch creation
- Strip "CU-" prefix from task ID if provided
- Never start coding without user approval
- Handle all errors gracefully with clear messages
- Follow ClickUp naming convention exactly
