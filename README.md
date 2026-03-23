# utils

Shared utilities, configs, and claude.md files for use across projects.

## Claude.md Files

- `ebay-buyer.claude.md` — eBay auction research assistant. Requires the official eBay MCP server (`@ebay/npm-public-api-mcp`).

## Usage

Clone this repo and symlink or include files into your project:

```bash
# Symlink into a project
ln -s ~/utils/ebay-buyer.claude.md ~/my-project/ebay-buyer.claude.md

# Or reference directly in your project's CLAUDE.md
# See: https://docs.anthropic.com/en/docs/claude-code
```
