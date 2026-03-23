# GitHub MCP Usage Guidelines

You have access to the GitHub MCP server (`@modelcontextprotocol/server-github`) AND local git/gh CLI tools. Use each for what it's best at.

## When to Use the GitHub MCP Server

Use the MCP tools for **searching and discovery** — read-only operations where you're exploring across repos:

- `search_repositories` — find repos by topic, language, name
- `search_code` — search code across GitHub (not just local repo)
- `search_issues` — find issues/PRs across repos
- `search_users` — find GitHub users
- `get_file_contents` — read a file from a remote repo you haven't cloned
- `list_issues` / `get_issue` — browse issues on a repo
- `list_pull_requests` / `get_pull_request` — browse PRs
- `get_pull_request_files` / `get_pull_request_comments` — review PR details

## When to Use git / gh CLI Instead

Use local CLI tools for **doing actual work** — anything that modifies state or involves the local repo:

### git (local repo operations)
- `git status`, `git diff`, `git log` — inspecting local state
- `git add`, `git commit` — staging and committing
- `git push`, `git pull` — syncing with remote
- `git branch`, `git checkout`, `git merge` — branch management
- `git stash`, `git rebase` — local history manipulation

### gh (GitHub CLI for actions)
- `gh pr create` — creating pull requests
- `gh pr merge` — merging pull requests
- `gh pr review` — submitting reviews
- `gh issue create` — creating issues
- `gh issue close` — closing issues
- `gh pr comment` / `gh issue comment` — adding comments
- `gh release create` — creating releases
- `gh run view` / `gh run watch` — checking CI status
- `gh api` — any GitHub API call that modifies state

## Why This Split

- **MCP is great for search/discovery** — it's built for exploring and finding things across GitHub without needing a local clone
- **git/gh CLI is precise** — for work that matters (commits, PRs, merges), use the tools designed for it. They have better error handling, support hooks, handle auth consistently, and produce predictable results
- **Avoid MCP for writes** — don't use `push_files`, `create_or_update_file`, or `create_pull_request` MCP tools when you have a local clone. Use git + gh instead

## General Guidelines

- If you have the repo cloned locally, prefer local tools over MCP
- Use MCP to look at repos you haven't cloned
- Never use MCP to push to a repo you're working in locally — it will diverge from your local state
- For PR reviews, MCP is fine for reading; use `gh` for submitting reviews or comments
