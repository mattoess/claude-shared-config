# MCP Server Setup Guide

This directory contains templates and documentation for setting up Model Context Protocol (MCP) servers with Claude Code.

## Available MCP Servers

| Server | Purpose | Auth Method |
|--------|---------|-------------|
| [ClickUp](#clickup) | Task management, tickets, project tracking | API Token |
| [Sentry](#sentry) | Error monitoring, debugging, issue analysis | OAuth |

---

## Quick Setup

### Sentry (Recommended - Remote Server)

Sentry uses a hosted remote MCP server with OAuth authentication:

```bash
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
```

After running this command:
1. Restart Claude Code
2. You'll be prompted to authenticate via OAuth
3. Log in with your Sentry account
4. The MCP tools will be available

**What you get:**
- 16+ tools for Sentry issue analysis
- Integration with Seer (AI-powered root cause analysis)
- Query issues, errors, and performance data
- Get AI-generated fix recommendations

### ClickUp (Local Server)

ClickUp uses a local npx-based MCP server:

1. Get your ClickUp API token from: https://app.clickup.com/settings/apps
2. Get your Team ID from any ClickUp URL: `https://app.clickup.com/{TEAM_ID}/...`
3. Add to your Claude config:

```bash
# Set your API token as environment variable
export CLICKUP_API_TOKEN="pk_YOUR_TOKEN_HERE"
```

Then add the MCP server configuration to `~/.claude.json`:

```json
{
  "mcpServers": {
    "clickup": {
      "command": "npx",
      "args": ["-y", "@hauptsache.net/clickup-mcp@latest"],
      "env": {
        "CLICKUP_API_KEY": "${CLICKUP_API_TOKEN}",
        "CLICKUP_TEAM_ID": "YOUR_TEAM_ID",
        "CLICKUP_MCP_MODE": "write",
        "MAX_IMAGES": "16",
        "MAX_RESPONSE_SIZE_MB": "4"
      }
    }
  }
}
```

---

## Verifying MCP Servers

After setup, verify your MCP servers are working:

```bash
# List configured MCP servers
claude mcp list

# Check server status (in Claude Code)
/mcp
```

---

## Troubleshooting

### Sentry MCP

**"Authentication required" error:**
- The OAuth flow should start automatically
- If not, remove and re-add: `claude mcp remove sentry && claude mcp add --transport http sentry https://mcp.sentry.dev/mcp`

**"No organizations found":**
- Ensure your Sentry account has access to at least one organization
- Check that your account permissions include API access

### ClickUp MCP

**"Invalid API key" error:**
- Verify `CLICKUP_API_TOKEN` is set in your environment
- Tokens start with `pk_` for personal tokens

**"Team not found" error:**
- Double-check your `CLICKUP_TEAM_ID` from a ClickUp URL
- Ensure your API token has access to that team

---

## Template Files

- `clickup.template.json` - ClickUp MCP configuration template
- `sentry.template.json` - Sentry MCP configuration template (for reference)

---

## Resources

- [Claude Code MCP Documentation](https://docs.anthropic.com/en/docs/claude-code/mcp)
- [Sentry MCP Server](https://docs.sentry.io/product/sentry-mcp/)
- [ClickUp API Documentation](https://clickup.com/api)
- [MCP Specification](https://modelcontextprotocol.io/)
