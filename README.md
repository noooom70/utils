# utils

Shared utilities, configs, and claude.md files for use across projects.

## MCP Configs

Each MCP we use (not build) gets a directory under `mcps/` with three files:

```
mcps/
└── <mcp-name>/
    ├── setup.md          # How to install, get credentials, configure
    ├── claude.md         # How Claude should use it (symlink into projects)
    └── permissions.json  # Recommended allow/deny/ask rules
```

### Available MCPs

| MCP | Purpose | Server Package |
|-----|---------|----------------|
| [ebay](mcps/ebay/) | Auction research, item comparison, price analysis | `@ebay/npm-public-api-mcp` |
| [github](mcps/github/) | Repo search & discovery (use git/gh CLI for actual work) | `@modelcontextprotocol/server-github` |
| [obsidian](mcps/obsidian/) | Vault search, reading/writing notes (WSL + Windows) | `@fazer-ai/mcp-obsidian` |

## Usage

To use an MCP's claude.md in a project, symlink it:

```bash
ln -s ~/utils/mcps/ebay/claude.md ~/my-project/ebay.claude.md
ln -s ~/utils/mcps/github/claude.md ~/my-project/github.claude.md
```

Or reference it from your project's CLAUDE.md.

## Permissions

Each MCP includes a `permissions.json` with recommended rules:

- **allow** — safe to auto-approve (reads, searches)
- **deny** — block entirely (destructive ops, or things better done via CLI)
- **ask** — prompt for confirmation each time (writes, comments, state changes)

To apply, merge the permissions into your `~/.claude/settings.local.json` under `permissions`.

## Adding a New MCP

1. Create `mcps/<name>/setup.md` with credentials, install, and config steps
2. Create `mcps/<name>/claude.md` with usage instructions for Claude
3. Create `mcps/<name>/permissions.json` with allow/deny/ask rules
4. Add it to the table above
