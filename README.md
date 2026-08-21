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

### Marketplace (when listed)

Install **Bootstrapware Importer** from the Cursor marketplace, then set the `BSW_SECRET` plugin variable under **Plugins → Configure**.

### Local / from this repo

1. Clone this repository.
2. Copy or symlink it into Cursor's local plugins folder as `bootstrapware-importer`:

```text
# Windows (PowerShell)
New-Item -ItemType Junction -Path "$env:USERPROFILE\.cursor\plugins\local\bootstrapware-importer" -Target "D:\Projects\bootstrapware-cursor"

# macOS / Linux
ln -s /path/to/bootstrapware-cursor ~/.cursor/plugins/local/bootstrapware-importer
```

3. Reload Cursor (or restart).
4. Open **Plugins → Bootstrapware Importer → Configure** and set `BSW_SECRET`.

### Manual MCP only (without the plugin)

Mint a **test secret** at https://app.bootstrapware.co/importer/keys, then merge into `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "bootstrapware-importer": {
      "type": "http",
      "url": "https://importer.bootstrapware.co/mcp",
      "headers": {
        "Authorization": "Bearer bsw_test_sec_PASTE_YOUR_KEY"
      }
    }
  }
}
```

Prefer the Keys page **Add to Cursor** button when available.

## First-run checklist

1. Create a **test** secret (`bsw_test_sec_…`) in the dashboard.
2. Set plugin variable `BSW_SECRET` (or paste into `mcp.json` as above).
3. Confirm MCP tools appear (e.g. `list_importers`, `list_capabilities`).
4. Smoke prompt:

```text
Using bootstrapware-importer MCP, list my importers and create a draft named "Cursor plugin smoke" with email (required) and name fields, then publish it.
```

## Plugin layout

```text
.cursor-plugin/plugin.json   # manifest + BSW_SECRET variable
mcp.json                     # HTTP MCP with Bearer ${BSW_SECRET}
skills/bootstrapware-importer/SKILL.md
assets/logo.png
```

## Security

- Never commit secrets.
- Never put secret keys in browser/client code.
- Never send spreadsheet file contents or parsed rows to Bootstrapware or through MCP tools.

## Marketplace submit

Submit this public repo at https://cursor.com/marketplace/publish once you are ready for review. Do not claim a marketplace listing until Cursor approves it.
