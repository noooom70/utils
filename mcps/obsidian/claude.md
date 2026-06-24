# Obsidian MCP Usage Guidelines

You have access to the `obsidian-fs` MCP server (`@bitbonsai/mcpvault`) — a
filesystem-based server that reads and writes the Obsidian vault's `.md` files
directly via `/mnt/c/...`. It does **not** talk to the Obsidian app or its REST
API, so it works whether or not Obsidian is running.

Tool names are namespaced `mcp__obsidian-fs__<tool>`.

## Available Tools (15)

### Reading
- `read_note` — read a note; returns `{fm, content}` (frontmatter parsed out separately from the body)
- `read_multiple_notes` — read several notes in one call
- `list_directory` — list files/folders under a vault path
- `get_frontmatter` — read just a note's frontmatter
- `get_notes_info` — metadata (size, dates, etc.) for one or more notes
- `get_vault_stats` — note/folder counts for the whole vault
- `list_all_tags` — every tag used in the vault

### Searching
- `search_notes` — BM25 full-text search over note content (and optionally frontmatter)

### Writing
- `write_note` — create, overwrite, append, or prepend; takes a `frontmatter` object separately from `content`
- `patch_note` — targeted insert/append/prepend into an existing note
- `update_frontmatter` — modify a note's frontmatter
- `manage_tags` — add/remove tags on a note

### Organizing / Deleting
- `move_note` — move or rename a note (updates inbound links)
- `move_file` — move or rename any vault file
- `delete_note` — delete a note; `trashMode` chooses permanent / vault `.trash` / system trash. Requires `confirmPath` == `path` as a guard.

## When to Use It

- **Reading notes** for context — research notes, project docs, learnings
- **Searching** the vault for information relevant to the current task
- **Writing** structured notes — project logs, summaries, research findings
- **Maintaining** the `Claude Code/` learnings folder (see the global CLAUDE.md)

## When NOT to Use It

- Don't use it as a database — it's a note-taking vault
- Don't bulk-create dozens of files — be purposeful
- Don't overwrite files without reading them first

## How to Use It Well

- **Search before creating** — check if a note already exists before making a new one
- **Append, don't overwrite** — use `patch_note` (or `write_note` in append/prepend mode) when adding to existing notes
- **Frontmatter is separate** — pass frontmatter via the `frontmatter` object; don't hand-write a `---` fence into `content`
- **Respect vault structure** — `list_directory` first to understand the organization before creating files in new locations
- **Keep formatting consistent** — use Markdown, match the style of existing notes in the vault

## What This Server Can't Do

Because it bypasses Obsidian's own engine, there's **no** Dataview / JsonLogic
querying, no periodic-notes endpoints, and no Obsidian command execution. Its
BM25 `search_notes` covers ordinary read/write/search. If you ever need those
Obsidian-native features, that requires the REST-API server instead — see
`setup.md`.
