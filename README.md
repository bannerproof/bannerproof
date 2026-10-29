# Bannerproof — cookie-consent audits for AI assistants and CI

[Bannerproof](https://bannerproof.app) checks a website the way a real first-time visitor from the EU sees it, and reports:

- trackers, analytics/ad tags and tracking cookies that load **before consent**;
- whether the cookie banner has **“Reject all” on the first layer**, and what **still fires after Reject**;
- **Google Consent Mode v2** state per signal (`ad_storage`, `analytics_storage`, `ad_user_data`, `ad_personalization`);
- **IAB TCF** consent string before a choice and after Reject;
- **Global Privacy Control** (informational in the EU);
- cookies the site sets that its own **cookie policy doesn’t mention**;
- third-party embeds that leak visitor data before consent (Google Fonts, YouTube, Maps, reCAPTCHA).

Every finding comes with the exact request as evidence, a concrete fix (13 languages) and the legal reference (ePrivacy Art. 5(3), GDPR, national law). Works with any consent platform — Cookiebot, Usercentrics, OneTrust, Complianz, CookieYes, Borlabs, iubenda or a custom banner.

This repository holds the public pieces: the MCP server listing, connection examples and a GitHub Action. The service itself runs at bannerproof.app.

---

## 1. MCP server (Claude, ChatGPT, Cursor, …)

Remote MCP server, Streamable HTTP, OAuth 2.0:

```
https://api.bannerproof.app/mcp
```

A free quick check works without an account (`https://api.bannerproof.app/mcp/free`). Full findings, monitoring, audits and evidence exports use a Bannerproof account — sign in with OAuth when your client asks, or use an API token from [Settings](https://bannerproof.app/settings).

| Client | How to connect |
|---|---|
| Claude (claude.ai / Desktop) | Settings → Connectors → find **Bannerproof** in the directory, or add a custom connector with the URL above |
| Claude Code | `claude mcp add --transport http bannerproof https://api.bannerproof.app/mcp` |
| Cursor | see [`examples/cursor-mcp.json`](examples/cursor-mcp.json) |
| Cline | see [`llms-install.md`](llms-install.md) |
| Other clients | add an MCP server with the URL above (OAuth), or send `Authorization: Bearer <API token>` |

Step-by-step guide: [bannerproof.app/help/connect-claude-chatgpt](https://bannerproof.app/help/connect-claude-chatgpt)

**Tools**

| Tool | What it does | Account |
|---|---|---|
| `quick_check` | yes/no + score for any public site | free |
| `explain_finding` | what a finding means, why it matters, how to fix it, legal basis | free |
| `get_capabilities` | what the server checks and what each tool does | free |
| `get_findings` | full findings with evidence, fixes and legal references | yes |
| `scan_domain` | fresh scan of a monitored domain | yes |
| `compare_scans` | what changed since the previous scan | yes |
| `get_evidence` | evidence pack (every request with phase and time, cookies, consent signals) as CSV/JSON | yes |
| `get_report_link` | private report link to send to a client or developer | yes |
| `audit_domain` / `audit_domains` | audit any site, or up to 10 at once (1 audit credit each; failed scans are refunded) | yes |
| `get_audits_status` | status of several audits at once | yes |
| `list_domains`, `add_domain`, `scan_history`, `account_info`, `get_upgrade_link` | account helpers | yes |

The server is listed in the [official MCP Registry](https://registry.modelcontextprotocol.io) as `app.bannerproof/bannerproof` ([`server.json`](server.json)).

---

## 2. Deploy check (CI)

Fail a pipeline when a release breaks cookie consent. The domain must be monitored in your Bannerproof account; create an API token in [Settings](https://bannerproof.app/settings) and store it as a secret.

### GitHub Action

```yaml
- uses: bannerproof/bannerproof@v1
  with:
    domain: example.com
    api-token: ${{ secrets.BANNERPROOF_TOKEN }}
    fail-on: new   # "new" = only issues this release added (default), "any" = any issue
```

Full workflow: [`examples/github-workflow.yml`](examples/github-workflow.yml).

### Any other CI

```bash
curl -fsS -X POST https://api.bannerproof.app/api/ci/check \
  -H "X-Api-Token: $BANNERPROOF_TOKEN" -H "Content-Type: application/json" \
  -d '{"domain":"example.com","failOn":"new"}'
```

| HTTP status | Meaning |
|---|---|
| 200 | pass |
| 422 | fail — the JSON body lists `new_issues` |
| 202 | still scanning — poll the `poll` URL from the body |
| 429 | the domain was scanned less than 5 minutes ago (`Retry-After` header) |
| 502 | the site couldn't be scanned |

### Webhook

In Settings you can also add a webhook: after every scan of a monitored domain Bannerproof POSTs a `scan.completed` event (score, new and resolved issues) signed with HMAC-SHA256 — `X-Bannerproof-Signature: sha256=<hex of HMAC(secret, "<X-Bannerproof-Timestamp>.<body>")>`.

---

Bannerproof is an automated technical check of what a site sends at scan time — not legal advice.

Support: support@bannerproof.app
