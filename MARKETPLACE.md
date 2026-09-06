# Marketplace publish checklist

Public source: https://github.com/BootstrapWare/bootstrapware-cursor

Use this after the GitHub repo is public and you are ready for Cursor review. **Do not claim a marketplace listing is live until Cursor approves it.**

## Listing copy (paste into https://cursor.com/marketplace/publish)

- **Name:** Bootstrapware
- **Source:** https://github.com/BootstrapWare/bootstrapware-cursor
- **Homepage:** https://bootstrapware.co
- **Short description:** Configure Bootstrapware Importer, Feedback, and Chat from Cursor via MCP (OAuth Connect).
- **Longer description:**

```
Bootstrapware MCP servers for Cursor: Importer (CSV/Excel schemas), Feedback (board config), and Chat (app config). Prefer OAuth Connect (URL-only mcp.json). Secret-key Bearer auth is a fallback.

Never send spreadsheet rows, feedback post bodies, chat message bodies, or file bytes through MCP. Tools manage configuration only.

Importer MCP: https://importer.bootstrapware.co/mcp
Feedback MCP: https://feedback.bootstrapware.co/mcp
Chat MCP: https://chat.bootstrapware.co/mcp
Keys: https://app.bootstrapware.co/{importer|feedback|chat}/keys
```

## 1. Confirm the public repo

- [x] Repo is `BootstrapWare/bootstrapware-cursor`
- [x] Default branch is `main`
- [x] `LICENSE` is MIT
- [x] `.cursor-plugin/plugin.json`, `mcp.json`, skills (importer + feedback + chat), and `assets/logo.png` are present
- [x] README does not claim the marketplace listing is already live

## 2. Local smoke

- [ ] Local plugin junction works; Connect OAuth for MCP servers
- [ ] Importer tools: `list_importers`, `list_capabilities`
- [ ] Feedback tools: `list_boards`, `list_capabilities`
- [ ] Chat tools: `list_apps`, `list_capabilities`, `get_install_snippet`

## 3. Submit to Cursor marketplace

1. Open https://cursor.com/marketplace/publish
2. Submit **Bootstrapware** pointing at https://github.com/BootstrapWare/bootstrapware-cursor
3. Wait for Cursor review / approval
4. Only then mention the listing on marketing / docs
