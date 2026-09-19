---
name: bootstrapware-importer
description: Configure and integrate Bootstrapware CSV/Excel importers (React package, hosted config, and MCP tools). Use when the user asks about @bootstrapware/importer, hosted importer fields, publishable keys, or MCP importer management.
---

# Bootstrapware Importer

## What this product does

Client-side CSV / TSV / XLSX import UI. Parsing, mapping, validation, preview, duplicate detection, and error export run in the browser.

**Never send file contents, parsed rows, or filenames to Bootstrapware.** The customer's `onComplete` callback receives normalized objects. Send those to the customer's own backend.

Hosted mode only fetches published field configuration and reports non-content session metadata.

## Install the React package

```bash
pnpm add @bootstrapware/importer
```

## Local schema (free)

```tsx
import { Importer } from "@bootstrapware/importer";
import "@bootstrapware/importer/styles.css";

<Importer
  fields={[
    { key: "email", label: "Email", type: "email", required: true },
    { key: "name", label: "Name", type: "string", required: true },
  ]}
  duplicateKey="email"
  onComplete={async (rows) => {
    await fetch("/api/customers/import", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ rows }),
    });
  }}
/>
```

## Hosted configuration

Publishable keys may appear in the browser. Secret keys must stay server-side. Never mint live or secret keys via MCP. For a test publishable key, call `ensure_test_publishable`.

```tsx
<Importer
  importerId="imp_..."
  publishableKey={process.env.NEXT_PUBLIC_BSW_IMPORTER_PUBLISHABLE_KEY}
  onComplete={(rows) => {
    void fetch("/api/customers/import", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ rows }),
    });
  }}
/>
```

Optional `apiBaseUrl` defaults to `https://importer.bootstrapware.co`.

Path B: `list_capabilities` → `create_importer` → `update_draft` (`fields` required; `name` optional) → `publish_importer` → `ensure_test_publishable` (if `isNew: true`, add `envLine` to `.env.local`; if `isNew: false` and env is empty, open https://app.bootstrapware.co/importer/keys) → `get_install_snippet`.

## Hosted MCP (this plugin)

Endpoint: `https://importer.bootstrapware.co/mcp`

**Preferred:** OAuth Connect: URL-only config, click Connect in Cursor, sign in at https://app.bootstrapware.co and Allow. No secret in `mcp.json`.

**Fallback:** secret key (`bsw_test_sec_` / `bsw_live_sec_`) as `Authorization: Bearer` (mint at https://app.bootstrapware.co/importer/keys).

**Never put spreadsheet file contents or row payloads into MCP tools.**

### Tools

- `list_importers`
- `get_importer`
- `create_importer`
- `update_draft` (does not publish; name optional, fields required)
- `publish_importer`
- `get_published_config`
- `get_install_snippet`
- `ensure_test_publishable`
- `list_capabilities`

### Dashboard-only (do not invent MCP tools)

Live/secret API key mint/revoke, webhooks, importer delete, branding, billing. Test publishable: `ensure_test_publishable`. Prefer `list_capabilities` and the dashboard at https://app.bootstrapware.co.

### Publish vs live entitlement

Publishing stores hosted config. Live `bsw_live_pub_` fetch still needs a paid Importer plan ($19.99). Test publishable keys work without that.

Local `fields` schema remains free forever.

## Field types

`string` | `number` | `date` | `email` | `enum` | `boolean` | `url`

`enum` requires `enumValues`.

Optional `aliases` is a string array of extra spreadsheet header names used during auto-map (in addition to `key` and `label`).

Optional `unique: true` on a field rejects duplicate values for that field within the file.

## Security

- Do not put `bsw_live_sec_` or `bsw_test_sec_` in client code.
- Do not upload spreadsheet contents to Bootstrapware.
- `onComplete` is the data path.

## Docs

- Product: https://bootstrapware.co/importer
- Agents: https://bootstrapware.co/importer/docs/agents
- Keys: https://app.bootstrapware.co/importer/keys
