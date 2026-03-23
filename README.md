# utils

Shared utilities, configs, and claude.md files for use across projects.

## MCP Configs

Each MCP we use (not build) gets a directory under `mcps/` with two files:

```
mcps/
└── <mcp-name>/
    ├── setup.md      # How to install, get credentials, configure
    └── claude.md     # How Claude should use it (symlink into projects)
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

## Adding a New MCP

1. Create `mcps/<name>/setup.md` with credentials, install, and config steps
2. Create `mcps/<name>/claude.md` with usage instructions for Claude
3. Add it to the table above
