# Licensy MCP

**Healthcare provider licensing data for AI agents.**
Verify U.S. medical licenses — status, expiration, and audit-grade source screenshots — from every state board, in real time, from inside any MCP-compatible agent.

<p>
  <a href="https://mcp.licensy.ai">🌐 mcp.licensy.ai</a> ·
  <a href="https://licensy.ai/mcp">📖 Docs</a> ·
  <a href="https://licensy.ai/pricing">💳 Pricing</a> ·
  <a href="mailto:support@licensy.ai">✉️ Support</a>
</p>

> Real-time verification across all **50 states + DC**, sourced directly from state medical boards and refreshed every 24 hours. No scraping to maintain, no per-board integrations to build — one tool call.

---

## What you can do

| | Capability | Tool |
|---|---|---|
| 🔎 | **Find a provider's licenses** — every state a physician is licensed in, by NPI | `list_physician_licenses` |
| ✅ | **Verify a license** — current status, expiration, and an is-active answer | `verify_license` · `get_license_status` |
| 🧾 | **Retrieve the official source** — signed, audit-grade screenshot of the board page | `get_screenshot_url` |
| 📚 | **Verify in bulk** — a whole roster in one parallel call | `bulk_verify` |
| ⚖️ | **Surface disciplinary signals** *(beta)* — suspensions, revocations, surrenders | `search_disciplinary_actions` |
| 🕰️ | **Point-in-time lookup** *(beta)* — "was this provider licensed on 2024-08-12?" | `get_license_history` |
| 🔔 | **Monitor changes** *(coming soon)* — webhook on status change, expiry, or new sanction | `subscribe_to_changes` |

---

## Quickstart

Licensy MCP is a **remote** server — nothing to install or self-host. Point your agent at `https://mcp.licensy.ai/mcp` and authenticate with your API key.

### 1. Get your API key

Grab your key from your [Licensy dashboard](https://licensy.ai/dashboard) → **Settings → API Keys**. It looks like `ck_…`. The free tier needs no card.

### 2. Add it to your agent

<details open>
<summary><b>Claude Desktop</b></summary>

Edit `claude_desktop_config.json` (Settings → Developer → Edit Config):

```json
{
  "mcpServers": {
    "licensy": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.licensy.ai/mcp", "--header", "Authorization:${AUTH}"],
      "env": { "AUTH": "Bearer ck_your_key_here" }
    }
  }
}
```

> The `Authorization:${AUTH}` split (no space) is intentional — it works around `mcp-remote` splitting header args on spaces. Restart Claude Desktop after saving.
</details>

<details>
<summary><b>Cursor</b></summary>

Add to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "licensy": {
      "url": "https://mcp.licensy.ai/mcp",
      "headers": { "Authorization": "Bearer ck_your_key_here" }
    }
  }
}
```
</details>

<details>
<summary><b>Any MCP client (Streamable HTTP)</b></summary>

```
Transport: streamable-http
URL:       https://mcp.licensy.ai/mcp
Header:    Authorization: Bearer ck_your_key_here
```
</details>

### 3. Ask

> "Is Dr. Jane Smith, NPI 1234567890, currently licensed in California?"

---

## Example prompts

```text
Verify this physician — NPI 1234567890 — in Texas.
Find every active license for NPI 1234567890.
Retrieve the official source document for that California license.
Check my roster of 40 NPIs for any expired or lapsed licenses.
Was NPI 1234567890 licensed in New York on 2024-08-12?
Are there any disciplinary actions on file for NPI 1234567890?
```

---

## Tools

Every tool returns structured JSON with `field` descriptions the model reads directly.

### `verify_license`
Single physician + state → `status`, `expiration_date`, `license_active`. Paid tiers add `license_number`, `issuance_date`, `board_source`, `last_refreshed_at`, and a signed `screenshot_url`.
**Input:** `npi` (preferred) or `license_number`, plus `state` (2-letter or full name).

### `list_physician_licenses`
One NPI → all licenses on file, one per state, each with status + expiration.

### `get_license_status`
The minimal-token companion to `verify_license` — returns just the status string. Built for tight agent loops and routing logic.

### `get_screenshot_url` · *paid*
A signed, 15-minute-TTL URL to the state board page used to verify the license — primary-source evidence for audit, credentialing, and insurance files.

### `bulk_verify` · *paid*
Up to 100 (Pro) / 1,000 (Enterprise) `(NPI, state)` pairs in a single parallelized call, returned in input order with a status summary.

### `search_disciplinary_actions` · *paid, beta*
Suspensions, revocations, surrenders, probation, and reprimands surfaced from board status text. (Underlying PDF final-orders are on the roadmap.)

### `get_license_history` · *paid, beta*
Point-in-time status for a past date. Returns best-available data with a `coverage_start` marker; full append-only history is rolling out.

### `subscribe_to_changes` / `unsubscribe` · *paid, coming soon*
Register a webhook for `status_change`, `expiration_warning_60d`, and `new_disciplinary_action` events. Registration is live today and your subscriptions persist; **automated delivery is rolling out** — [contact us](mailto:support@licensy.ai) for early access.

---

## Resources

Read-only context your agent can pull without spending a verification call:

| Resource | What |
|---|---|
| `licensy://states/{code}` | State board metadata: official URL, credential types, known quirks |
| `licensy://credentials/{type}` | Canonical metadata for MD, DO, PA, APRN, RN, CNP, CRNA, CNS, CNM |
| `licensy://schema/license` | JSON schema of the canonical license record |
| `licensy://changelog` | Recent platform events: board outages, refresh failures, schema changes |

---

## Tiers

| | Free | Pro | Enterprise |
|---|---|---|---|
| Verify status + expiration | ✅ | ✅ | ✅ |
| Multi-state license list | ✅ | ✅ | ✅ |
| License number, dates, board source | | ✅ | ✅ |
| Signed source screenshots | | ✅ | ✅ |
| Bulk verify | | 100/call | 1,000/call |
| Disciplinary + history + webhooks | | ✅ | ✅ |
| Rate limit | 100/day | 10,000/day | unlimited |

Tiers are enforced server-side. A free-tier call to a paid tool returns a structured `tier_upgrade_required` response (with an upgrade link) — never a hard error — so your agent can surface the upgrade naturally. See [pricing](https://licensy.ai/pricing).

---

## How it works

Licensy maintains live verification pipelines against all 51 U.S. medical-board jurisdictions and normalizes every board's idiosyncratic status vocabulary into one clean schema (`active`, `expired`, `suspended`, `revoked`, …). The MCP server is a thin, audited layer over that platform — it never invents data, and when a board is briefly unreachable it returns a retryable `service_unavailable`, never a false "not licensed."

---

## Links

- **Endpoint:** `https://mcp.licensy.ai/mcp`
- **Discovery card:** `https://mcp.licensy.ai/.well-known/mcp-server`
- **Docs:** https://licensy.ai/mcp
- **Support:** support@licensy.ai

*Licensy is a data platform for provider licensing. Verification data is sourced from public state-board records; see [terms](https://licensy.ai/terms).*
