# Bootstrapware Cursor plugin

Configure hosted CSV/Excel importers from Cursor via MCP, plus an agent skill for integrating `@bootstrapware/importer`.

| | |
| --- | --- |
| Homepage | https://bootstrapware.co/importer |
| MCP | `https://importer.bootstrapware.co/mcp` |
| Keys | https://app.bootstrapware.co/importer/keys |
| License | MIT |

Spreadsheet contents never leave the customer's browser. MCP tools manage **importer configuration only** (fields, publish), not row payloads.

## Install

### Preferred: OAuth Connect

Install the plugin (marketplace or local), open **Tools & MCP**, and click **Connect** on Bootstrapware Importer. Sign in at `app.bootstrapware.co` and approve access. No secret key in `mcp.json`.

Or use **Add to Cursor (OAuth)** on https://app.bootstrapware.co/importer/keys.

URL-only config:

```json
{
  "mcpServers": {
    "bootstrapware-importer": {
      "type": "http",
      "url": "https://importer.bootstrapware.co/mcp"
    }
  }
}
```

### Secret key fallback

If OAuth Connect is unavailable, mint a test secret at Keys and paste Bearer auth into `mcp.json` (see Keys page **Add to Cursor (secret fallback)**).

### Local / from this repo

1. Clone this repository.
2. Junction/symlink into `~/.cursor/plugins/local/bootstrapware-importer`.
3. Reload Cursor and Connect via OAuth.

## First-run checklist

1. Install plugin or URL-only MCP config.
2. Click **Connect** → sign in → Allow.
3. Confirm tools (`list_importers`, `list_capabilities`).
4. Smoke prompt:

```text
Using bootstrapware-importer MCP, list my importers and create a draft named "Cursor OAuth smoke" with email (required) and name fields, then publish it.
```

5. Optional: revoke the connection on Keys → Active Cursor connections and confirm tools fail until Connect again.

## Plugin layout

```text
.cursor-plugin/plugin.json
mcp.json                     # HTTP MCP URL only (OAuth)
skills/bootstrapware-importer/SKILL.md
assets/logo.png
```

## Security

- Prefer OAuth; never put secrets in client/browser code.
- Never send spreadsheet file contents or parsed rows to Bootstrapware or through MCP tools.
