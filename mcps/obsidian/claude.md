# Obsidian MCP Usage Guidelines

You have access to the Obsidian MCP server (`@fazer-ai/mcp-obsidian`) which connects to the user's Obsidian vault via the Local REST API plugin.

## Available Tools

### Reading
- `obsidian_get_file` — read a specific file by path
- `obsidian_get_active` — read the currently open/active file
- `obsidian_get_periodic` — read a periodic note (daily, weekly, monthly)
- `obsidian_list_vault_root` — list files/folders at vault root
- `obsidian_list_vault_directory` — list files/folders in a directory
- `obsidian_status` — check connection status

### Searching
- `obsidian_simple_search` — full-text search across the vault
- `obsidian_search_json_logic` — structured search using JsonLogic
- `obsidian_search_dataview` — query using Dataview plugin syntax (if installed)

### Writing
- `obsidian_post_file` — create a new file
- `obsidian_put_file` — overwrite an entire file
- `obsidian_patch_file` — append, prepend, or insert into a file
- `obsidian_post_active` / `obsidian_put_active` / `obsidian_patch_active` — modify the active file
- `obsidian_post_periodic` / `obsidian_put_periodic` / `obsidian_patch_periodic` — modify periodic notes

### Other
- `obsidian_delete_file` / `obsidian_delete_active` / `obsidian_delete_periodic` — delete files
- `obsidian_open_file` — open a file in Obsidian's UI
- `obsidian_execute_command` — run an Obsidian command
- `obsidian_get_commands` — list available commands

## When to Use Obsidian MCP

- **Reading notes** for context — research notes, project docs, meeting notes
- **Searching** the vault for information relevant to the current task
- **Writing** structured notes — project logs, summaries, research findings
- **Daily notes** — appending to today's daily note with updates or logs

## When NOT to Use Obsidian MCP

- Don't use it as a database — it's a note-taking tool
- Don't bulk-create dozens of files — be purposeful
- Don't overwrite files without reading them first

## How to Use It Well

- **Search before creating** — check if a note already exists before making a new one
- **Append, don't overwrite** — use `obsidian_patch_file` with append/prepend when adding to existing notes
- **Respect vault structure** — list directories first to understand the organization before creating files in new locations
- **Use the active file** — if the user has a file open, they probably want you to work with it. Check `obsidian_get_active` when relevant
- **Keep formatting consistent** — use Markdown, match the style of existing notes in the vault

## Prerequisites

- Obsidian must be running on Windows for any tools to work
- If tools return "fetch failed," tell the user to check that Obsidian is open and the Local REST API plugin is enabled
