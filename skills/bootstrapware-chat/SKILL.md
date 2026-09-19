---
name: bootstrapware-chat
description: Configure and integrate Bootstrapware Chat (React package, BYO/Hosted modes, MCP tools, install algorithm). Use when the user asks about @bootstrapware/chat, in-app messaging, Hosted Chat, or MCP app management.
---

# Bootstrapware Chat

Embeddable private 1:1 and group messaging (max 20) inside an authenticated product. Not Stream, Sendbird, Slack, or a helpdesk.

**Never send message body or file bytes through MCP.** Never mint live or secret API keys via MCP. For a test publishable key, call `ensure_test_publishable`. Never invent `user.id` for production.

## Install algorithm (do this in order)

1. Read this skill. Call `list_capabilities` on Chat MCP before inventing tools.
2. If the user only needs a working UI: Path A. Do not invent keys.
3. If they need live Hosted or BYO config: Path B or C.
4. Wire the **real** session id from the host app.
5. For a test publishable key, call `ensure_test_publishable`.

### Path A — local, zero keys

```bash
pnpm add @bootstrapware/chat
```

```tsx
import { Chat, createLocalAdapter } from "@bootstrapware/chat";
import "@bootstrapware/chat/styles.css";

<Chat
  user={{ id: session.user.id, name: session.user.name, avatarUrl: session.user.image }}
  people={workspaceMembers.map((m) => ({ id: m.id, name: m.name, avatarUrl: m.image }))}
  adapter={createLocalAdapter({ storageKey: "demo-chat" })}
/>
```

`people` is display and picker only. Optional `launcher` opens a corner bubble with an unread badge. Nudge with `--bsw-ch-launcher-right` / `--bsw-ch-launcher-bottom`.

Bodies store plain text. The widget renders a tiny subset (`**bold**`, `_italic_`, `` `code` ``, simple lists) without HTML.

### Path B — Hosted

1. Confirm Cursor MCP `bootstrapware-chat` at `https://chat.bootstrapware.co/mcp` (OAuth URL-only). If disconnected, tell the human to click **Add to Cursor (OAuth)** on https://app.bootstrapware.co/chat/keys
2. `list_capabilities` → `create_app` → `update_draft` (toggles + `allowedOrigins` for localhost and production; `name` is optional) → `publish_app`
3. `ensure_test_publishable`. If `isNew: true`, add `envLine` to `.env.local`. If `isNew: false` and env is empty, open https://app.bootstrapware.co/chat/keys
4. `get_install_snippet`. `appId` is real. Use the env publishable key, never invent one.
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

`list_apps`, `get_app`, `create_app`, `update_draft` (name optional), `publish_app`, `get_published_config`, `get_install_snippet`, `ensure_test_publishable`, `list_capabilities`.

Dashboard-only: live/secret keys, webhooks, delete, branding, billing, Hosted inbox (including Team send), message search. Test publishable: `ensure_test_publishable`.

`create_app` / `publish_app` return `nextSteps[]`.

## Identity and origins

Host asserts `user={{ id, name?, avatarUrl? }}`. Optional `people` for picker, titles, and avatars. No anonymous chat. Optional `authorToken` minted with a **secret** key via `POST /api/v1/author-tokens` when `requireAuthorToken` is true.

Set `allowedOrigins` on the draft (localhost + production) before live. Default `["*"]` is for first-run only.

## Pricing

- Free: local + test keys
- BYO $9.99: live config; you store messages and files
- Hosted $19.99: we store messages and files (1 GB included). Optional Chat storage add-on $9.99 raises the cap to 10 GB (not a product, not stack-discounted)
- Cancel Hosted: freeze writes, JSON export 30 days, then delete Hosted messages and objects
- Customer moderates in Inbox; this is not a helpdesk

## Security

- Secret keys server-side only
- Publishable keys may appear in the browser
- MCP/webhooks: ids only
