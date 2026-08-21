# Marketplace publish checklist

Public source: https://github.com/BootstrapWare/bootstrapware-cursor

Use this after the GitHub repo is public and you are ready for Cursor review. **Do not claim a marketplace listing is live until Cursor approves it.**

## 1. Confirm the public repo

- [ ] Repo is public under `BootstrapWare/bootstrapware-cursor`
- [ ] Default branch is `main`
- [ ] `LICENSE` is MIT
- [ ] `.cursor-plugin/plugin.json`, `mcp.json`, skill, and `assets/logo.png` are present
- [ ] README does not claim the marketplace listing is already live

## 2. Local smoke (already done once)

- [ ] Local plugin junction/symlink works under `~/.cursor/plugins/local/bootstrapware-importer`
- [ ] `BSW_SECRET` (test secret) configured
- [ ] MCP tools appear (`list_importers`, `list_capabilities`, etc.)

## 3. Submit to Cursor marketplace

1. Open https://cursor.com/marketplace/publish
2. Submit **Bootstrapware Importer** pointing at https://github.com/BootstrapWare/bootstrapware-cursor
3. Wait for Cursor review / approval
4. Only after approval, update product docs to say the marketplace install is available

## 4. Until marketplace is approved

Prefer the dashboard Keys page **Add to Cursor** button for MCP setup:

https://app.bootstrapware.co/importer/keys

Optional: clone/symlink this repo as a local Cursor plugin (see README).
