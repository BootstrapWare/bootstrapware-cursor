# Bootstrapware Cursor plugin

Configure **Importer** and **Feedback** from Cursor via MCP, plus agent skills for `@bootstrapware/importer` and `@bootstrapware/feedback`.

| | Importer | Feedback |
| --- | --- | --- |
| Homepage | https://bootstrapware.co/importer | https://bootstrapware.co/feedback |
| MCP | `https://importer.bootstrapware.co/mcp` | `https://feedback.bootstrapware.co/mcp` |
| Keys | https://app.bootstrapware.co/importer/keys | https://app.bootstrapware.co/feedback/keys |
| License | MIT | MIT |

One plugin, two products. MCP tools manage **configuration only** — never spreadsheet rows or feedback post bodies.

## Install

### Preferred: OAuth Connect

Install the plugin, open **Tools & MCP**, and click **Connect** on each Bootstrapware MCP server. Sign in at `app.bootstrapware.co` and approve access. No secret key in `mcp.json`.

Or use **Add to Cursor (OAuth)** on each product’s Keys page.

URL-only config (both products):

```json
{
  "mcpServers": {
    "bootstrapware-importer": {
      "type": "http",
      "url": "https://importer.bootstrapware.co/mcp"
    },
    "bootstrapware-feedback": {
      "type": "http",
      "url": "https://feedback.bootstrapware.co/mcp"
    }
  }
}
```

### Secret key fallback

Mint a test secret on the product Keys page and paste Bearer auth into `mcp.json` if OAuth Connect is unavailable.

### Local / from this repo

1. Clone this repository.
2. Junction/symlink into `~/.cursor/plugins/local/bootstrapware` (or keep the historical `bootstrapware-importer` folder name).
3. Reload Cursor and Connect via OAuth.

## First-run checklist

1. Install plugin or URL-only MCP config.
2. Click **Connect** on Importer and/or Feedback → sign in → Allow.
3. Confirm tools (`list_capabilities` on each server).
4. Smoke prompts:

```text
Using bootstrapware-importer MCP, list my importers and create a draft named "Cursor OAuth smoke" with email (required) and name fields, then publish it.
```

```text
Using bootstrapware-feedback MCP, list my boards and create a draft board named "Cursor Feedback smoke", then publish it.
```

5. Optional: revoke connections on Keys → Active Cursor connections.

## Plugin layout

```text
.cursor-plugin/plugin.json
mcp.json                     # HTTP MCP URLs only (OAuth)
skills/bootstrapware-importer/SKILL.md
skills/bootstrapware-feedback/SKILL.md
assets/logo.png
```

## Security

- Prefer OAuth; never put secrets in client/browser code.
- Never send spreadsheet contents, parsed rows, or feedback post title/body through MCP tools.
