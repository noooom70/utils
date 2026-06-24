# Obsidian MCP Server Setup for Claude Code (WSL + Windows)

Guide for connecting Claude Code (running in WSL) to an Obsidian vault on
Windows. The recommended server is **`@bitbonsai/mcpvault`** — a
filesystem-based MCP server that reads/writes the vault's `.md` files directly.
No Obsidian plugin, no running Obsidian app, no `cmd.exe` bridge.

## Why filesystem over the REST-API approach

The REST-API alternative (`@fazer-ai/mcp-obsidian` + Obsidian's Local REST API
plugin) has two structural drawbacks a filesystem server avoids:

- The REST plugin returns an empty **HTTP 204** body on successful PUT/POST/DELETE;
  the server then calls `response.json()` on the empty body → **`Unexpected end
  of JSON input`** on operations that actually *succeeded*.
- It requires the Obsidian app **running on Windows**, the **Local REST API
  plugin** installed/configured (API key, port, self-signed cert), and a
  **`cmd.exe /c npx ...`** bridge because the REST API only listens on Windows
  localhost.

A filesystem server sidesteps all of it: it reads/writes the vault's `.md` files
directly via `/mnt/c/...`, returns structured JSON over stdio (no 204, no parse
error), and needs no plugin and no running Obsidian.

## Architecture

```
Claude Code (WSL) → npx @bitbonsai/mcpvault (WSL) → /mnt/c/.../Vault/*.md
```

Everything runs natively in WSL.

## Prerequisites

- WSL2 with **Node ≥20** (`node --version`). mcpvault declares `engines: node >=20`;
  it runs on 18 but emits `EBADENGINE` warnings — use Node 20+ LTS (e.g. via `nvm`).
- An Obsidian vault stored on the Windows side, reachable from WSL at
  `/mnt/c/Users/<you>/<Your Vault>`.
- The Obsidian app does **not** need to be running.

## Step 1: Register the MCP Server

Use the `claude mcp` CLI (preferred over hand-editing `~/.claude.json`):

```bash
claude mcp add obsidian-fs --scope user -- \
  npx -y @bitbonsai/mcpvault@latest "/mnt/c/Users/<you>/<Your Vault>"
```

The vault path is the Windows vault as seen from WSL (`C:\Users\you\Vault` →
`/mnt/c/Users/you/Vault`). Spaces in the path are fine — it's a single argv
element, not shell-split. `--scope user` makes it available in every project;
drop it for project-local scope.

This writes the following into `~/.claude.json` under `mcpServers`:

```jsonc
"obsidian-fs": {
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@bitbonsai/mcpvault@latest", "/mnt/c/Users/<you>/<Your Vault>"],
  "env": {}
}
```

Verify with `claude mcp get obsidian-fs`.

## Step 2: Allowlist the Tools

So the tools don't prompt, add them to `~/.claude/settings.json` →
`permissions.allow`. The simplest entry is a **server-level wildcard**,
`"mcp__obsidian-fs"`, which covers all 15 tools; or list
`mcp__obsidian-fs__<tool>` per tool for finer control (see `permissions.json` in
this folder for a per-tool reference). To keep deletes prompting, add
`"mcp__obsidian-fs__delete_note"` to `permissions.ask` — it also has a built-in
`confirmPath` guard regardless.

## Step 3: Restart and Verify

1. Exit Claude Code (`/exit`) — MCP config is read at session start.
2. Relaunch Claude Code.
3. Run `/mcp`; `obsidian-fs` should show **connected with 15 tools**.
4. Smoke test: `get_vault_stats` should return note/folder counts. A `write_note`
   to a scratch file should return `Successfully wrote note: ...` with **no** JSON
   error; clean up with `delete_note` (pass `confirmPath` == `path`; optionally
   `trashMode: "local"` to send it to the vault `.trash`).

You can also smoke-test the server **before** registering it — spawn it over
stdio and send an `initialize` + `tools/list` JSON-RPC pair; a healthy launch
returns `serverInfo` and the 15 tool names.

## Tools (15)

`read_note`, `read_multiple_notes`, `write_note` (overwrite/append/prepend),
`patch_note`, `delete_note` (permanent / `.trash` / system-trash via
`trashMode`), `move_note`, `move_file`, `list_directory`, `search_notes` (BM25),
`get_frontmatter`, `update_frontmatter`, `manage_tags`, `list_all_tags`,
`get_notes_info`, `get_vault_stats`.

mcpvault returns frontmatter separately from body (`write_note` takes a
`frontmatter` object; `read_note` returns `{fm, content}`) — don't hand-write a
`---` fence into the content.

## Tradeoffs / when you'd want the REST approach instead

Filesystem access bypasses Obsidian's own engine, so you lose **Dataview /
JsonLogic queries, periodic-notes endpoints, and Obsidian command execution**.
mcpvault provides its own **BM25 `search_notes`** instead, which covers ordinary
read/write/search. If you specifically need those Obsidian-native features, use
the REST-API server below.

### Alternative: REST-API server (`@fazer-ai/mcp-obsidian`)

Requires Obsidian running on Windows + the Local REST API plugin, bridged
through `cmd.exe`. Setup:

1. **Install Node on Windows** (`winget install OpenJS.NodeJS.LTS`), then reboot
   so PATH propagates to all processes (including WSL). Verify from WSL with
   `cmd.exe /c "npx --version"`.
2. **Install & enable** the **Local REST API** community plugin in Obsidian.
   Note its **API key** and **HTTPS port** (default `27124`), and trust its
   self-signed certificate (install to *Trusted Root Certification Authorities*
   for the Current User).
3. **Set Windows env vars** (PowerShell), then reopen WSL so `cmd.exe` inherits
   them:
   ```powershell
   setx OBSIDIAN_API_KEY "YOUR_API_KEY"
   setx OBSIDIAN_PORT "27124"
   setx OBSIDIAN_PROTOCOL "https"
   setx NODE_TLS_REJECT_UNAUTHORIZED "0"
   ```
   `NODE_TLS_REJECT_UNAUTHORIZED=0` is needed because the plugin uses a
   self-signed cert. Verify with `cmd.exe /c "echo %OBSIDIAN_API_KEY%"`.
4. **Register** the server — everything after `/c` must stay a **single string**;
   verify it in `~/.claude.json`:
   ```bash
   claude mcp add obsidian -- cmd.exe /c "npx -y @fazer-ai/mcp-obsidian \"C:\Users\you\Vault\""
   ```
5. **Restart** Claude Code, run `/mcp`, and call `obsidian_status` — it should
   return `"authenticated": true`.

**Known quirk:** write/delete tools error with `Unexpected end of JSON input`
even on success (the HTTP 204 issue above) — verify the operation actually
landed with a follow-up list/read.

**Troubleshooting** ("connected" but tools return "fetch failed"): confirm
Obsidian is running, the plugin is enabled, all four env vars are set
(`cmd.exe /c "echo %VAR%"`), the REST API responds
(`cmd.exe /c "curl -sk https://localhost:27124"`), and Node is on the Windows
PATH. If env vars print as `%NAME%`, close and reopen the WSL terminal.
