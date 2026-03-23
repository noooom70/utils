# GitHub MCP Server Setup

## MCP Server

Official GitHub MCP: `@modelcontextprotocol/server-github`
- Repo: https://github.com/modelcontextprotocol/servers/tree/main/src/github
- Requires: Node.js

## 1. Get a GitHub Personal Access Token

1. Go to https://github.com/settings/tokens
2. Generate a **fine-grained** or **classic** token
3. Scopes needed: `repo`, `read:org`, `read:user` (at minimum)
4. Copy the token

## 2. Add to Claude Code Settings

Add to `~/.claude.json` under `mcpServers`:

```json
"github": {
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-github"],
  "env": {
    "GITHUB_PERSONAL_ACCESS_TOKEN": "YOUR_TOKEN"
  }
}
```

## 3. Link the Claude.md

Symlink the usage instructions into your project:

```bash
ln -s ~/utils/mcps/github/claude.md ~/my-project/github.claude.md
```
