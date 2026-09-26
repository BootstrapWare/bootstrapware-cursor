---
name: bootstrapware-comments
description: Configure and integrate Bootstrapware Comments (React package, BYO/Hosted modes, and MCP tools). Use when the user asks about @bootstrapware/comments, discussions on a resource, publishable keys for Comments, or MCP app management.
---

# Bootstrapware Comments

## What this product does

Embeddable discussions on a host-owned resource: threads and flat replies, open or resolved.

**Modes**

- **Local / free:** `createLocalAdapter` (demo and development).
- **BYO ($9.99):** you store discussions via `CommentsAdapter`. Bootstrapware hosts app config only.
- **Hosted ($19.99):** Bootstrapware stores threads and comments. Customer moderates in the dashboard.

**Never send comment bodies, resource content, or customer directories through MCP.** MCP manages app configuration only. Never mint live or secret API keys via MCP. For a test publishable key, call `ensure_comment_test_publishable`.

## Install

```bash
pnpm add @bootstrapware/comments
```

```tsx
import { Comments, createLocalAdapter } from "@bootstrapware/comments";
import "@bootstrapware/comments/styles.css";
```

## Identity

Bootstrapware does **not** authenticate discussion visitors. Your app asserts opaque `user={{ id, name?, avatarUrl? }}`. The widget prop is `user`, not `viewerId`.

`scope` (`tenantKey`, `resourceType`, `resourceId`) is a widget prop filled from a record your server already authorized. Those strings are not MCP arguments and are not proof of access.

Optional `authorToken` is minted on your server with a Comments **secret** key (`POST https://comments.bootstrapware.co/api/v1/author-tokens`). Never mint it in the browser or via MCP. Pass `renewAuthorToken` so the widget can refresh it. Mention lookup stays in host callbacks (`searchMentionCandidates`, `resolveUsers`). Do not send that directory to MCP.

## Local

```tsx
<Comments
  user={{ id: session.user.id, name: session.user.name }}
  scope={{ tenantKey: workspace.id, resourceType: "task", resourceId: task.id }}
  adapter={createLocalAdapter({ storageKey: "demo-comments" })}
/>
```

## Hosted

```tsx
<Comments
  appId="cma_..."
  publishableKey={process.env.NEXT_PUBLIC_BSW_COMMENTS_PUBLISHABLE_KEY}
  user={currentUser}
  scope={{ tenantKey: workspace.id, resourceType: "task", resourceId: task.id }}
  authorToken={authorTokenFromYourServer}
  renewAuthorToken={renewFromYourServer}
/>
```

`apiBaseUrl` defaults to `https://comments.bootstrapware.co`.

Path B: `list_comment_capabilities` → `create_comment_app` → `update_comment_draft` (toggles and `allowedOrigins`; `name` optional) → `publish_comment_app` → `ensure_comment_test_publishable` (put `envLine` in `.env.local`; the full test publishable is returned every time) → `get_comment_install_snippet`. Embed with the real session user id, never `"user_1"`.

`update_comment_draft` `config` accepts only `emptyState`, `mentionsEnabled`, `resolveEnabled`, `requireAuthorToken`, `bodyMax`, and `allowedOrigins`. `emptyState` is a short prompt (200 characters max). `bodyMax` is a length limit (1–10000), not a comment body. Do not pass comment text, resource titles, tenant or resource ids, mention directories, or file bytes.

## Hosted MCP (this plugin)

Endpoint: `https://comments.bootstrapware.co/mcp`

Server id: `bootstrapware-comments`

**Preferred:** OAuth Connect: URL-only config, Connect in Cursor, sign in at https://app.bootstrapware.co.

**Fallback:** secret key as `Authorization: Bearer` (mint at https://app.bootstrapware.co/comments/keys).

### Tools

App config only: `list_comment_apps`, `get_comment_app`, `create_comment_app`, `update_comment_draft` (name optional), `publish_comment_app`, `get_comment_published_config`, `get_comment_install_snippet`, `ensure_comment_test_publishable`, `list_comment_capabilities`. No comment bodies, resource content, or customer directories.

Dashboard-only: live and secret key mint, webhooks, app delete, branding, billing, Hosted discussion browser, export, purge. Test publishable: `ensure_comment_test_publishable`.

`create_comment_app` and `publish_comment_app` return `nextSteps[]`. Call `list_comment_capabilities` before using any other tool name.

## Pricing

- Free: local + test keys
- BYO $9.99: live config
- Hosted $19.99: we store discussions
- Cancel Hosted: freeze writes, export 30 days, then delete Hosted data
- Customer moderates; Bootstrapware provides tools only

## Security

- Secret keys server-side only
- Plain text comments; encode on render
- Publishable keys and `allowedOrigins` do not prove the end user may read or write
