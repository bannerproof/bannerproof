# Installing the Bannerproof MCP server

Bannerproof is a **remote** MCP server (Streamable HTTP). Nothing needs to be cloned, built or installed — only the client configuration below.

## 1. Free quick check — no account, no key

Add this to the MCP settings file (in Cline: `cline_mcp_settings.json`):

```json
{
  "mcpServers": {
    "bannerproof": {
      "type": "streamableHttp",
      "url": "https://api.bannerproof.app/mcp/free"
    }
  }
}
```

Available without an account: `quick_check` (yes/no + score for any public website), `get_quick_check_result`, `explain_finding`, `get_capabilities`.

Check that it works: ask the assistant to run `quick_check` for `example.com`.

## 2. Full access — Bannerproof account

Full findings with evidence, monitoring, audits and evidence exports need a Bannerproof account.

**Option A — OAuth (clients that support MCP OAuth):** use `https://api.bannerproof.app/mcp` as the URL and sign in when the client opens the browser.

**Option B — API token (works in every client):** the user creates a token at https://bannerproof.app/settings (section *API token*) and puts it in the header:

```json
{
  "mcpServers": {
    "bannerproof": {
      "type": "streamableHttp",
      "url": "https://api.bannerproof.app/mcp",
      "headers": {
        "Authorization": "Bearer <BANNERPROOF_API_TOKEN>"
      }
    }
  }
}
```

Never invent a token — ask the user for it. Without a valid token `https://api.bannerproof.app/mcp` answers `401`; in that case fall back to the free URL from step 1.

## Notes

- No environment variables, Node, Python or Docker are required.
- `audit_domain` / `audit_domains` spend 1 audit credit per site from the user's account; failed scans are refunded automatically.
- Tool reference and client-by-client guide: [README](README.md) and https://bannerproof.app/help/connect-claude-chatgpt
