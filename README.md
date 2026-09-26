# Bootstrapware Cursor plugin

Configure **Importer**, **Feedback**, **Chat**, **Onboard**, and **Comments** from Cursor via MCP, plus agent skills for `@bootstrapware/importer`, `@bootstrapware/feedback`, `@bootstrapware/chat`, `@bootstrapware/onboard`, and `@bootstrapware/comments`.

| | Importer | Feedback | Chat | Onboard | Comments |
| --- | --- | --- | --- | --- | --- |
| Homepage | https://bootstrapware.co/importer | https://bootstrapware.co/feedback | https://bootstrapware.co/chat | https://bootstrapware.co/onboard | https://bootstrapware.co/comments |
| MCP | `https://importer.bootstrapware.co/mcp` | `https://feedback.bootstrapware.co/mcp` | `https://chat.bootstrapware.co/mcp` | `https://onboard.bootstrapware.co/mcp` | `https://comments.bootstrapware.co/mcp` |
| Keys | https://app.bootstrapware.co/importer/keys | https://app.bootstrapware.co/feedback/keys | https://app.bootstrapware.co/chat/keys | https://app.bootstrapware.co/onboard/keys | https://app.bootstrapware.co/comments/keys |
| License | MIT | MIT | MIT | MIT | MIT |

One plugin, five products. MCP tools manage **configuration only** — never spreadsheet rows, feedback post bodies, chat message bodies, host context/flags/facts, progress payloads, comment bodies, resource content, or customer directories.

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
    },
    "bootstrapware-onboard": {
      "type": "http",
      "url": "https://onboard.bootstrapware.co/mcp"
    },
    "bootstrapware-comments": {
      "type": "http",
      "url": "https://comments.bootstrapware.co/mcp"
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
2. Click **Connect** on Importer, Feedback, Chat, Onboard, and/or Comments → sign in → Allow.
3. Confirm tools (`list_capabilities` on Importer, Feedback, Chat, and Onboard; `list_comment_capabilities` on Comments).
4. Smoke prompts:

```text
Using bootstrapware-importer MCP, list my importers and create a draft named "Cursor OAuth smoke" with email (required) and name fields, then publish it.
```

```text
Using bootstrapware-feedback MCP, list my boards and create a draft board named "Cursor Feedback smoke", then publish it.
```

```text
Using bootstrapware-chat MCP, list my apps and create a draft named "Cursor Chat smoke", set allowedOrigins to localhost, then publish it. Call ensure_test_publishable and get_install_snippet.
```

```text
Using bootstrapware-onboard MCP, list my flows and create a draft named "Cursor Onboard smoke" from the SaaS first-run template, set allowedOrigins to localhost, then publish it. Call ensure_test_publishable and get_install_snippet.
```

```text
Using bootstrapware-comments MCP, list my comment apps and create a draft named "Cursor Comments smoke", set allowedOrigins to localhost, then publish it. Call ensure_comment_test_publishable and get_comment_install_snippet.
```

5. Optional: revoke connections on Keys → Active Cursor connections.

## Plugin layout

```text
.cursor-plugin/plugin.json
mcp.json                     # HTTP MCP URLs only (OAuth)
skills/bootstrapware-importer/SKILL.md
skills/bootstrapware-feedback/SKILL.md
skills/bootstrapware-chat/SKILL.md
skills/bootstrapware-onboard/SKILL.md
skills/bootstrapware-comments/SKILL.md
assets/logo.png
```

## Security

- Prefer OAuth; never put secrets in client/browser code.
- Never send spreadsheet contents, parsed rows, feedback post title/body, chat message bodies, host context/flags/facts, progress payloads, comment bodies, resource content, or customer directories through MCP tools.
