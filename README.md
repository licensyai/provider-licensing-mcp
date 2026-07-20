# Licensy MCP

**Healthcare licensing infrastructure — over the Model Context Protocol.**

Nationwide **physician and physician assistant** licensing data, standardized from official public sources across all 50 states + DC, delivered to AI applications through MCP.

<p>
  <a href="https://mcp.licensy.ai">mcp.licensy.ai</a> ·
  <a href="https://licensy.ai/mcp">Documentation</a> ·
  <a href="mailto:support@licensy.ai">Contact</a>
</p>

> Integrate once instead of building and maintaining separate integrations against dozens of state licensing board websites. Licensy aggregates and normalizes licensing information into a consistent schema, preserves primary-source screenshots where available, and refreshes monitored records daily.

---

## What it provides

An MCP interface to Licensy's provider-licensing data layer. Tools return a normalized license record — not a per-state HTML page you have to parse.

| Capability | Tool |
|---|---|
| List a provider's licenses across states, by NPI | `list_physician_licenses` |
| Look up a license's current status + expiration | `verify_license` · `get_license_status` |
| Retrieve the preserved primary-source screenshot for a record | `get_screenshot_url` |
| Look up many `(NPI, state)` pairs in one call | `bulk_verify` |
| Surface disciplinary signals from board status text *(beta)* | `search_disciplinary_actions` |
| Point-in-time status for a past date *(beta)* | `get_license_history` |
| Monitor a set of providers for changes *(coming soon)* | `subscribe_to_changes` |

**Current coverage:** physician (MD/DO) and physician assistant (PA) licenses, all 50 states + DC. Additional provider categories are on the roadmap, not yet available.

---

## Connect

Licensy MCP is a **remote** server — nothing to install or host. Point your MCP client at `https://mcp.licensy.ai/mcp` and authenticate with an API key.

**Get access:** request API / MCP access at [licensy.ai/mcp](https://licensy.ai/mcp). Your key comes from the Licensy dashboard → **Settings → API Keys** (starts with `ck_`).

<details open>
<summary><b>Claude Desktop</b></summary>

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
> The `Authorization:${AUTH}` split (no space) works around `mcp-remote` splitting header args on spaces. Restart Claude Desktop after saving.
</details>

<details>
<summary><b>Cursor</b> (<code>~/.cursor/mcp.json</code>)</summary>

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

---

## Example prompts

```text
List every state where NPI 1234567890 holds a license.
What is the current status of NPI 1234567890's Texas license?
Retrieve the source screenshot for that California license.
Check this roster of NPIs for any expired or lapsed licenses.
Are there disciplinary signals on file for NPI 1234567890?
```

---

## Tools

Every tool returns structured JSON; field descriptions are included so the model can use the output directly.

- **`verify_license`** — one provider + state → normalized `status`, `expiration_date`, `license_active`. Paid access adds `license_number`, `issuance_date`, `board_source`, `last_refreshed_at`, and a signed `screenshot_url`. Input: NPI (preferred) or license number, plus state.
- **`list_physician_licenses`** — all licenses on file for one NPI, one per state.
- **`get_license_status`** — minimal status-only response for tight agent loops.
- **`get_screenshot_url`** *(paid)* — a signed, time-limited URL to the preserved screenshot of the official record, as supporting source context.
- **`bulk_verify`** *(paid)* — many `(NPI, state)` pairs in one call, returned in input order with a status summary.
- **`search_disciplinary_actions`** *(paid, beta)* — disciplinary signals surfaced from board status text.
- **`get_license_history`** *(paid, beta)* — best-available point-in-time status for a past date.
- **`subscribe_to_changes` / `unsubscribe`** *(paid, coming soon)* — register a webhook for change events; delivery is rolling out.

## Resources

Read-only reference context (no lookup consumed):

| Resource | Contents |
|---|---|
| `licensy://states/{state_code}` | State board metadata: official URL, credential types, known quirks |
| `licensy://credentials/{credential_type}` | Metadata for supported credential types (MD, DO, PA) |
| `licensy://schema/license` | JSON schema of the normalized license record |
| `licensy://changelog` | Recent platform events (board outages, refresh notes, schema changes) |

---

## How it works

Licensy maintains data pipelines against U.S. medical-board sources, normalizes each board's differing status vocabulary into one schema (`active`, `expired`, `suspended`, `revoked`, …), preserves primary-source screenshots where available, and refreshes monitored records daily. The MCP server is a thin, audited layer over that data platform — when a source is briefly unreachable it returns a retryable `service_unavailable`, never a false "not licensed."

## Links

- **Endpoint:** `https://mcp.licensy.ai/mcp`
- **Discovery card:** `https://mcp.licensy.ai/.well-known/mcp-server`
- **Documentation:** https://licensy.ai/mcp
- **Contact:** support@licensy.ai

---

*Licensy AI provides licensing information derived from official public sources. Availability, completeness, and update timing may vary by jurisdiction and source. Licensy does not replace credentialing, primary-source verification requirements, legal review, or an organization's own decision-making; customers remain responsible for how the information is used in their workflows.*
