# Bootstrapware Cursor plugin

Configure **Importer**, **Feedback**, and **Chat** from Cursor via MCP, plus agent skills for `@bootstrapware/importer`, `@bootstrapware/feedback`, and `@bootstrapware/chat`.

| | Importer | Feedback | Chat |
| --- | --- | --- | --- |
| Homepage | https://bootstrapware.co/importer | https://bootstrapware.co/feedback | https://bootstrapware.co/chat |
| MCP | `https://importer.bootstrapware.co/mcp` | `https://feedback.bootstrapware.co/mcp` | `https://chat.bootstrapware.co/mcp` |
| Keys | https://app.bootstrapware.co/importer/keys | https://app.bootstrapware.co/feedback/keys | https://app.bootstrapware.co/chat/keys |
| License | MIT | MIT | MIT |

One plugin, three products. MCP tools manage **configuration only** — never spreadsheet rows, feedback post bodies, chat message bodies, or file bytes.

## Install

### Preferred: OAuth Connect

Install the plugin, open **Tools & MCP**, and click **Connect** on each Bootstrapware MCP server. Sign in at `app.bootstrapware.co` and approve access. No secret key in `mcp.json`.

Or use **Add to Cursor (OAuth)** on each product’s Keys page.

URL-only config (all products):

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
    },
    "bootstrapware-chat": {
      "type": "http",
      "url": "https://chat.bootstrapware.co/mcp"
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
2. Click **Connect** on Importer, Feedback, and/or Chat → sign in → Allow.
3. Confirm tools (`list_capabilities` on each server).
4. Smoke prompts:

```text
Using bootstrapware-importer MCP, list my importers and create a draft named "Cursor OAuth smoke" with email (required) and name fields, then publish it.
```

```text
Using bootstrapware-feedback MCP, list my boards and create a draft board named "Cursor Feedback smoke", then publish it.
```

```text
Using bootstrapware-chat MCP, list my apps and create a draft named "Cursor Chat smoke", set allowedOrigins to localhost, then publish it. If no publishable key is in env, stop and tell me to mint one on the Keys page.
```

5. Optional: revoke connections on Keys → Active Cursor connections.

## Plugin layout

```text
.cursor-plugin/plugin.json
mcp.json                     # HTTP MCP URLs only (OAuth)
skills/bootstrapware-importer/SKILL.md
skills/bootstrapware-feedback/SKILL.md
skills/bootstrapware-chat/SKILL.md
assets/logo.png
```

## Security

- Prefer OAuth; never put secrets in client/browser code.
- Never send spreadsheet contents, parsed rows, feedback post title/body, chat message bodies, or file bytes through MCP tools.
