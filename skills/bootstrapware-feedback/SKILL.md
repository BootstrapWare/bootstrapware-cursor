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

**Never send post title/body through MCP.** MCP manages board configuration only. Never mint live or secret API keys via MCP. For a test publishable key, call `ensure_test_publishable`.

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
  publishableKey={process.env.NEXT_PUBLIC_BSW_FEEDBACK_PUBLISHABLE_KEY}
  user={currentUser}
/>
```

`apiBaseUrl` defaults to `https://feedback.bootstrapware.co`.

Path B: `list_capabilities` → `create_board` → `update_draft` (toggles; `name` optional) → `publish_board` → `ensure_test_publishable` (put `envLine` in `.env.local`; the full test publishable is returned every time) → `get_install_snippet`. Embed with the real session user id, never `"user_1"`.

## Hosted MCP (this plugin)

Endpoint: `https://feedback.bootstrapware.co/mcp`

**Preferred:** OAuth Connect: URL-only config, Connect in Cursor, sign in at https://app.bootstrapware.co.

**Fallback:** secret key as `Authorization: Bearer` (mint at https://app.bootstrapware.co/feedback/keys).

### Tools

Board config only: `list_boards`, `get_board`, `create_board`, `update_draft` (name optional), `publish_board`, `get_published_config`, `get_install_snippet`, `ensure_test_publishable`, `list_capabilities`. No post bodies.

Dashboard-only: live/secret keys, webhooks, delete, branding, billing, Hosted inbox. Test publishable: `ensure_test_publishable`.

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
