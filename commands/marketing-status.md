Show the current state of the RGA Campaign marketing pipeline from ClickUp.

## Process

1. Read `clickup.json` for workspace config and `.env` for the `CLICKUP_API_KEY`

2. Fetch all tasks from the RGA Campaign list (ID: 901409083855):

```bash
curl -s -X GET "https://api.clickup.com/api/v2/list/901409083855/task?include_closed=true" \
  -H "Authorization: $CLICKUP_API_KEY" | python3 -c "
import json, sys
d = json.load(sys.stdin)
groups = {}
for t in d.get('tasks', []):
    status = t['status']['status']
    groups.setdefault(status, []).append(t)

order = ['in progress', 'planned', 'to do', 'blocked', 'done', 'complete', 'cancelled']
for status in order:
    tasks = groups.get(status, [])
    if not tasks:
        continue
    print(f'\n## {status.title()} ({len(tasks)})')
    for t in tasks:
        tags = ', '.join(tag['name'] for tag in t.get('tags', []))
        tag_str = f'  [{tags}]' if tags else ''
        print(f'  - {t[\"name\"]}{tag_str}')
"
```

3. Present a clean summary:

```
## RGA Campaign Pipeline

### In Progress (N)
- [task names]

### Planned (N)
- [task names]

### To Do (N)
- [task names]

### Recently Completed (N)
- [task names]

View full list: https://app.clickup.com/9010149796/v/li/901409083855
```

4. If the user asks about a specific task, fetch its details:

```bash
curl -s -X GET "https://api.clickup.com/api/v2/task/{task_id}" \
  -H "Authorization: $CLICKUP_API_KEY"
```

5. Offer next steps:
   - "Want to create new content? Run `/release-marketing`"
   - "Want to update a task status?" — offer to move tasks between statuses
