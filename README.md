# Claude Shared Config

Reusable Claude Code configuration for cross-project use. Add as a git submodule to share skills, commands, and MCP templates across all your projects.

## Quick Start

### 1. Add to Your Project

```bash
# Add as submodule
git submodule add https://github.com/mattoess/claude-shared-config.git

# Create symlinks to .claude directory
mkdir -p .claude
ln -s ../claude-shared-config/skills .claude/skills
ln -s ../claude-shared-config/commands .claude/commands
```

### 2. Create Project Config

Create `clickup.json` in your project root:

```json
{
  "workspace_id": "YOUR_WORKSPACE_ID",
  "space_id": "YOUR_SPACE_ID",
  "branch_naming": {
    "pattern": "CU-{task_id}_{description}_{developer_name}",
    "developer_name": "Your-Name"
  }
}
```

### 3. Configure MCP Server

Create `.mcp.json` in your project root (copy from `mcp/clickup.template.json`):

```json
{
  "mcpServers": {
    "clickup": {
      "command": "npx",
      "args": ["-y", "@hauptsache.net/clickup-mcp@latest"],
      "env": {
        "CLICKUP_API_KEY": "${CLICKUP_API_TOKEN}",
        "CLICKUP_TEAM_ID": "YOUR_TEAM_ID",
        "CLICKUP_MCP_MODE": "write"
      }
    }
  }
}
```

### 4. Set Environment Variable

Add to your `~/.zshrc` or `~/.bashrc`:

```bash
export CLICKUP_API_TOKEN="your-api-token"
```

## What's Included

### Skills

| Skill | Description |
|-------|-------------|
| `clickup/` | ClickUp integration via MCP server |
| `github/` | Git/GitHub operations with safety checks |
| `security/` | OWASP Top 10 security scanning |
| `sentry/` | Error monitoring and debugging via MCP |

### Commands

| Command | Description |
|---------|-------------|
| `/ticket <id>` | Start work on a ClickUp ticket |
| `/commit` | Generate conventional commit message |
| `/merge` | Safe merge operations |
| `/security-scan` | Run security checks |
| `/audit` | Dependency vulnerability audit |
| `/sentry-debug` | Analyze and debug Sentry errors |

## Updating

```bash
# Update submodule to latest
git submodule update --remote claude-shared-config
git add claude-shared-config
git commit -m "chore: Update claude-shared-config"
```

## Structure

```
claude-shared-config/
├── skills/
│   ├── clickup/skill.md      # ClickUp MCP integration
│   ├── github/skill.md       # Git operations
│   ├── security/             # Security skills
│   │   ├── skill.md          # Main security coordinator
│   │   ├── owasp-injection.md
│   │   ├── owasp-auth.md
│   │   └── owasp-access-control.md
│   └── sentry/skill.md       # Sentry MCP integration
├── commands/
│   ├── ticket.md
│   ├── commit.md
│   ├── merge.md
│   ├── security-scan.md
│   ├── audit.md
│   └── sentry-debug.md
├── mcp/
│   ├── README.md             # MCP setup guide
│   ├── clickup.template.json
│   └── sentry.template.json
├── templates/
│   ├── clickup.json.template
│   └── CLAUDE.md.template
└── README.md
```

## Project-Specific Overrides

Keep these files in your project (not in submodule):
- `clickup.json` - Project-specific ClickUp config
- `.mcp.json` - MCP server config with your credentials
- `CLAUDE.md` - Project-specific Claude instructions

## License

MIT
