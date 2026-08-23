# Marketplace publish checklist

Public source: https://github.com/BootstrapWare/bootstrapware-cursor

Use this after the GitHub repo is public and you are ready for Cursor review. **Do not claim a marketplace listing is live until Cursor approves it.**

## Listing copy (paste into https://cursor.com/marketplace/publish)

- **Name:** Bootstrapware Importer
- **Source:** https://github.com/BootstrapWare/bootstrapware-cursor
- **Homepage:** https://bootstrapware.co/importer
- **Short description:** Configure hosted CSV/Excel importers from Cursor via MCP (OAuth Connect).
- **Longer description:**

```
Bootstrapware Importer lets a coding agent create, draft, and publish hosted CSV / TSV / XLSX importer schemas from Cursor. Prefer OAuth Connect (URL-only MCP, no secret in mcp.json). Secret-key Bearer auth is a fallback.

Spreadsheet contents never leave the customer's browser. MCP tools manage configuration only: list_importers, get_importer, create_importer, update_draft, publish_importer, get_published_config, list_capabilities.

Dashboard for keys, webhooks, delete, branding, and billing: https://app.bootstrapware.co/importer/keys
```

## 1. Confirm the public repo

- [x] Repo is `BootstrapWare/bootstrapware-cursor`
- [x] Default branch is `main`
- [x] `LICENSE` is MIT
- [x] `.cursor-plugin/plugin.json`, `mcp.json`, skill, and `assets/logo.png` are present
- [x] README does not claim the marketplace listing is already live

## 2. Local smoke (already done once)

- [x] Local plugin junction/symlink works under `~/.cursor/plugins/local/bootstrapware-importer`
- [x] MCP tools appear (`list_importers`, `list_capabilities`, etc.)

## 3. Submit to Cursor marketplace

1. Open https://cursor.com/marketplace/publish
2. Submit **Bootstrapware Importer** pointing at https://github.com/BootstrapWare/bootstrapware-cursor
3. Wait for Cursor review / approval
4. Only after approval, update product docs to say the marketplace install is available

Submission itself requires a logged-in Cursor account. An agent cannot complete Cursor's review on your behalf.

## 4. Until marketplace is approved

Prefer the dashboard Keys page **Add to Cursor** button for MCP setup:

https://app.bootstrapware.co/importer/keys

Optional: clone/symlink this repo as a local Cursor plugin (see README).
