# eBay MCP Server Setup

## MCP Server

Official eBay MCP: `@ebay/npm-public-api-mcp`
- Repo: https://github.com/eBay/npm-public-api-mcp
- Requires: Node.js 22+

## 1. Get eBay Developer Credentials

1. Create an account at https://developer.ebay.com/
2. Go to **Application Access Keys**
3. Create a new application (Production keyset)
4. Note your **Client ID** (App ID) and **Client Secret** (Cert ID)
5. Account approval takes ~1 business day

## 2. Generate an Access Token

### App Token (search, item details, sold prices)

```bash
curl -s https://api.ebay.com/identity/v1/oauth2/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -u "YOUR_CLIENT_ID:YOUR_CLIENT_SECRET" \
  -d "grant_type=client_credentials&scope=https%3A%2F%2Fapi.ebay.com%2Foauth%2Fapi_scope"
```

This returns a token valid for ~2 hours. Save the `access_token` value.

### User Token (watch list access — optional, set up later)

Requires OAuth Authorization Code Grant flow:
1. Direct user to eBay consent URL
2. User approves, eBay redirects with auth code
3. Exchange auth code for refresh token
4. Refresh token lasts 18 months

Details: https://developer.ebay.com/api-docs/static/oauth-authorization-code-grant.html

## 3. Add to Claude Code Settings

Add to `~/.claude/settings.json` under `mcpServers`:

```json
"ebay-api": {
  "command": "npx",
  "args": ["-y", "@ebay/npm-public-api-mcp@latest"],
  "env": {
    "EBAY_CLIENT_TOKEN": "YOUR_ACCESS_TOKEN",
    "EBAY_API_ENV": "production"
  }
}
```

## 4. Token Refresh

The app token expires every ~2 hours. To refresh, re-run the curl command from step 2 and update `EBAY_CLIENT_TOKEN` in settings.

TODO: Consider a helper script to automate token refresh.

## 5. Link the Claude.md

Symlink the usage instructions into your project:

```bash
ln -s ~/utils/mcps/ebay/claude.md ~/my-project/ebay-buyer.claude.md
```

Or include it from your project's CLAUDE.md:
```markdown
See also: ~/utils/mcps/ebay/claude.md
```
