---
name: bootstrapware-chat
description: Configure and integrate Bootstrapware Chat (React package, BYO/Hosted modes, MCP tools, install algorithm). Use when the user asks about @bootstrapware/chat, in-app messaging, Hosted Chat, or MCP app management.
---

# Bootstrapware Chat

Embeddable private 1:1 and group messaging (max 20) inside an authenticated product. Not Stream, Sendbird, Slack, or a helpdesk.

**Never send message body or file bytes through MCP.** Never mint API keys via MCP. Never invent `user.id` for production.

## Install algorithm (do this in order)

1. Read this skill. Call `list_capabilities` on Chat MCP before inventing tools.
2. If the user only needs a working UI: Path A. Do not invent keys.
3. If they need live Hosted or BYO config: Path B or C.
4. Wire the **real** session id from the host app.
5. If no publishable key is in env, **stop and ask the human**.

### Path A — local, zero keys

```bash
pnpm add @bootstrapware/chat
```

```tsx
import { Chat, createLocalAdapter } from "@bootstrapware/chat";
import "@bootstrapware/chat/styles.css";

<Chat
  user={{ id: session.user.id, name: session.user.name }}
  adapter={createLocalAdapter({ storageKey: "demo-chat" })}
/>
```

### Path B — Hosted

1. Confirm Cursor MCP `bootstrapware-chat` at `https://chat.bootstrapware.co/mcp` (OAuth URL-only). If disconnected, tell the human to click **Add to Cursor (OAuth)** on https://app.bootstrapware.co/chat/keys
2. `list_capabilities` → `create_app` → `update_draft` (toggles + `allowedOrigins` for localhost and production) → `publish_app`
3. `get_install_snippet`. `appId` is real. The key is a placeholder (`YOUR_PUBLISHABLE_KEY`).
4. If env has no `bsw_test_pub_` / `bsw_live_pub_` value, **stop**. Ask the human to mint a test publishable key at https://app.bootstrapware.co/chat/keys
5. Embed with the real session user id, never `"user_1"` in production.

```tsx
<Chat
  appId="cha_..."
  publishableKey={process.env.NEXT_PUBLIC_BSW_CHAT_PUBLISHABLE_KEY}
  user={{ id: session.user.id, name: session.user.name }}
/>
```

`apiBaseUrl` defaults to `https://chat.bootstrapware.co`.

### Path C — BYO

Same MCP app config as Path B. Implement `ChatAdapter` on the customer backend (`uploadAttachment`, optional `subscribe` / `searchMessages`). Bodies and file bytes never go to Bootstrapware.

## MCP tools

`list_apps`, `get_app`, `create_app`, `update_draft`, `publish_app`, `get_published_config`, `get_install_snippet`, `list_capabilities`.

Dashboard-only: keys, webhooks, delete, branding, billing, Hosted inbox (including Team send), message search.

`create_app` / `publish_app` return `nextSteps[]`.

## Identity and origins

Host asserts `user={{ id, name? }}`. No anonymous chat. Optional `authorToken` minted with a **secret** key via `POST /api/v1/author-tokens` when `requireAuthorToken` is true.

Set `allowedOrigins` on the draft (localhost + production) before live. Default `["*"]` is for first-run only.

## Pricing

- Free: local + test keys
- BYO $9: live config; you store messages and files
- Hosted $19: we store messages and files (1 GB included). Optional Chat storage add-on $9 raises the cap to 10 GB (not a product, not stack-discounted)
- Cancel Hosted: freeze writes, JSON export 30 days, then delete Hosted messages and objects
- Customer moderates in Inbox; this is not a helpdesk

## Security

- Secret keys server-side only
- Publishable keys may appear in the browser
- MCP/webhooks: ids only
