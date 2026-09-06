---
name: bootstrapware-feedback
description: Configure and integrate Bootstrapware Feedback boards (React package, BYO/Hosted modes, and MCP tools). Use when the user asks about @bootstrapware/feedback, feature-request boards, publishable keys for Feedback, or MCP board management.
---

# Bootstrapware Feedback

## What this product does

Embeddable feature-request board: list, submit, vote, fixed statuses.

**Modes**

- **Local / free:** `createLocalAdapter` (demo and development).
- **BYO ($9.99):** you store posts via `FeedbackAdapter`. Bootstrapware hosts board config only.
- **Hosted ($19.99):** Bootstrapware stores posts/votes. Customer moderates in the dashboard.

**Never send post title/body through MCP.** MCP manages board configuration only.

## Install

```bash
pnpm add @bootstrapware/feedback
```

```tsx
import { Feedback, createLocalAdapter } from "@bootstrapware/feedback";
import "@bootstrapware/feedback/styles.css";
```

## Identity

Bootstrapware does **not** authenticate board visitors. Your app asserts opaque `user={{ id, name? }}`. Optional `authorToken` is reserved for a later signed assertion (unused in v1).

## Local

```tsx
<Feedback
  user={{ id: "user_1", name: "Ada" }}
  adapter={createLocalAdapter({ storageKey: "demo-feedback" })}
/>
```

## Hosted

```tsx
<Feedback
  boardId="brd_..."
  publishableKey="bsw_live_pub_..."
  user={currentUser}
/>
```

`apiBaseUrl` defaults to `https://feedback.bootstrapware.co`.

## Hosted MCP (this plugin)

Endpoint: `https://feedback.bootstrapware.co/mcp`

**Preferred:** OAuth Connect: URL-only config, Connect in Cursor, sign in at https://app.bootstrapware.co.

**Fallback:** secret key as `Authorization: Bearer` (mint at https://app.bootstrapware.co/feedback/keys).

### Tools

Board config only: `list_boards` / `list_importers`-style board tools, `get_published_config`, `list_capabilities`. No post bodies.

Dashboard-only: keys, webhooks, delete, branding, billing, Hosted inbox.

## Pricing

- Free: local + test keys
- BYO $9.99: live config
- Hosted $19.99: we store posts
- Cancel Hosted: freeze writes, export 30 days, then delete Hosted data
- Customer moderates; Bootstrapware provides tools only

## Security

- Secret keys server-side only
- Plain text posts; encode on render
- Stolen publishable key can forge `author.id` until `authorToken` exists
