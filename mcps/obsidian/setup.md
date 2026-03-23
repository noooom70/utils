# Obsidian MCP Server Setup for Claude Code (WSL + Windows 11)

Guide for connecting Claude Code (running in WSL) to Obsidian (running in Windows) via the `@fazer-ai/mcp-obsidian` MCP server and the Local REST API community plugin.

## Architecture

```
Claude Code (WSL) → cmd.exe → npx MCP server (Windows) → Local REST API plugin → Obsidian (Windows)
```

The MCP server must run on the Windows side (via `cmd.exe`) because:
- Obsidian's REST API listens on Windows `localhost`, which is not accessible from WSL
- Environment variables set on Windows don't pass through to WSL processes

## Prerequisites

- Windows 11 with WSL2
- Claude Code installed in WSL
- Obsidian installed on Windows with an active vault

## Step 1: Install Node.js on Windows

Node.js is needed on the **Windows side** (not just WSL) because the MCP server runs via `cmd.exe`.

Open PowerShell as Administrator:

```powershell
winget install OpenJS.NodeJS.LTS --accept-package-agreements --accept-source-agreements
```

**Reboot Windows** after installation so the PATH updates propagate to all processes including WSL.

Verify after reboot (from WSL):

```bash
cmd.exe /c "npx --version"
```

## Step 2: Install and Configure the Obsidian Local REST API Plugin

1. Open Obsidian
2. Go to **Settings → Community plugins → Browse**
3. Search for **"Local REST API"** (by Adam Coddington)
4. Click **Install**, then **Enable**
5. Go to **Settings → Local REST API**
6. Note the **API Key** — you'll need it in Step 3
7. Note the **HTTPS port** — default is `27124`
8. Download the **certificate** file (there should be a button/link in the plugin settings)
9. Double-click the downloaded certificate in Windows Explorer:
   - Click **Install Certificate**
   - Select **Current User**
   - Choose **Place all certificates in the following store**
   - Select **Trusted Root Certification Authorities**
   - Click **Finish**

## Step 3: Set Windows Environment Variables

Open PowerShell (regular, not WSL) and run these commands, replacing `YOUR_API_KEY` with the key from Step 2:

```powershell
setx OBSIDIAN_API_KEY "YOUR_API_KEY"
setx OBSIDIAN_PORT "27124"
setx OBSIDIAN_PROTOCOL "https"
setx NODE_TLS_REJECT_UNAUTHORIZED "0"
```

**Close and reopen your WSL terminal** after setting these so `cmd.exe` inherits the new variables.

Verify (from WSL):

```bash
cmd.exe /c "echo %OBSIDIAN_API_KEY%"
```

This should print your API key, not `%OBSIDIAN_API_KEY%`.

## Step 4: Register the MCP Server in Claude Code

From your project directory in WSL:

```bash
claude mcp add obsidian -- cmd.exe /c "npx -y @fazer-ai/mcp-obsidian \"C:\Users\YOUR_USERNAME\Your Obsidian Vault\""
```

Replace the vault path with your actual Windows path to the Obsidian vault.

**Important**: Claude Code's `mcp add` command splits arguments individually, but `cmd.exe /c` needs everything after `/c` as a single string. After running the command, verify the config is correct:

```bash
cat ~/.claude.json | python3 -c "import json,sys; d=json.load(sys.stdin); [print(json.dumps(v,indent=2)) for k,v in d.get('projects',{}).items() for name,srv in v.get('mcpServers',{}).items() if name=='obsidian']"
```

The `args` array should look like this — everything after `/c` as **one string**:

```json
{
  "type": "stdio",
  "command": "cmd.exe",
  "args": [
    "/c",
    "npx -y @fazer-ai/mcp-obsidian \"C:\\Users\\YOUR_USERNAME\\Your Obsidian Vault\""
  ],
  "env": {}
}
```

If the args are split into separate entries (e.g., `"/c", "npx", "-y", ...`), you'll need to manually edit `~/.claude.json` and join them into a single string as shown above.

## Step 5: Restart and Verify

1. Exit Claude Code (`/exit`)
2. Relaunch Claude Code in your project directory
3. Run `/mcp` to check the server status — it should show as connected with 25 tools
4. Ask Claude to run the `obsidian_status` tool — it should return `"authenticated": true`

## Troubleshooting Checklist

If the server shows "connected" but tools return "fetch failed":

| Check | How to verify |
|-------|---------------|
| Obsidian is running | Open it on Windows |
| Local REST API plugin is enabled | Settings → Community plugins |
| API key env var is set | `cmd.exe /c "echo %OBSIDIAN_API_KEY%"` from WSL |
| Port env var is set | `cmd.exe /c "echo %OBSIDIAN_PORT%"` should show `27124` |
| Protocol env var is set | `cmd.exe /c "echo %OBSIDIAN_PROTOCOL%"` should show `https` |
| TLS override is set | `cmd.exe /c "echo %NODE_TLS_REJECT_UNAUTHORIZED%"` should show `0` |
| REST API is responding | `cmd.exe /c "curl -sk https://localhost:27124"` should return JSON |
| Node.js is on Windows PATH | `cmd.exe /c "npx --version"` should return a version number |

If env vars show as `%VARIABLE_NAME%` instead of values, close your WSL terminal entirely and reopen it.

## Notes

- The MCP server is registered **per-project** in Claude Code. If you want it in multiple projects, run `claude mcp add` from each project directory, or add it with `-s user` for global access.
- Obsidian must be running on Windows whenever you want to use the MCP tools. Without it, the server connects but all tool calls fail.
- The `@fazer-ai/mcp-obsidian` package (v1.2.0+) is used instead of the original `mcp-obsidian` package, which does not properly register tools with Claude Code.
- The `NODE_TLS_REJECT_UNAUTHORIZED=0` setting disables TLS certificate verification for Node.js processes launched from `cmd.exe`. This is scoped to the MCP server process and is necessary because the Local REST API plugin uses a self-signed certificate.
