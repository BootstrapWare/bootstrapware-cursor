---
name: bootstrapware-onboard
description: Configure and integrate Bootstrapware Onboard (React package, BYO/Hosted modes, MCP tools, install algorithm). Use when the user asks about @bootstrapware/onboard, first-run setup checklists, Hosted Onboard, or MCP flow management.
---

# Bootstrapware Onboard

Embeddable first-run setup checklist for SaaS: required and optional steps, dependencies, eligibility, fact reconciliation, dismiss, and snooze. Userflow and similar products also offer MCP — Onboard is not unique in that regard.

**Never send host context, flags, facts, or progress payloads through MCP.** Never mint live or secret API keys via MCP. For a test publishable key, call `ensure_test_publishable`. Never invent `user.id` for production.

## Install algorithm (do this in order)

1. Read this skill.
2. If the user only needs a working UI: Path A. Do not invent keys. Do not wait for MCP.
3. If they need live Hosted or BYO config: Path B or C. Call `list_capabilities` on Onboard MCP before inventing tools.
4. Wire the **real** session id and `workspaceKey` from the host app.
5. For a test publishable key, call `ensure_test_publishable`.

### Path A — local, zero keys

```bash
pnpm add @bootstrapware/onboard
```

```tsx
import { Onboard, createLocalAdapter, SAAS_FIRST_RUN_ITEMS } from "@bootstrapware/onboard";
import "@bootstrapware/onboard/styles.css";

<Onboard
  user={{ id: session.user.id, name: session.user.name }}
  workspaceKey={workspace.id}
  context={{ role: membership.role, plan: workspace.plan, flags: featureFlags }}
  facts={{ hasBilling: billingConnected }}
  items={SAAS_FIRST_RUN_ITEMS}
  adapter={createLocalAdapter({ storageKey: "demo-onboard" })}
/>
```

Call `completeStep` from your app on real actions **and** pass `facts` so the widget reconciles on mount.

### Path B — Hosted

1. Confirm Cursor MCP `bootstrapware-onboard` at `https://onboard.bootstrapware.co/mcp` (OAuth URL-only). If disconnected, tell the human to click **Add to Cursor (OAuth)** on https://app.bootstrapware.co/onboard/keys
2. `list_capabilities` → `create_flow` (SaaS first-run items are already on the draft) → `update_draft` (`allowedOrigins` for localhost and production; only send `items` if you are changing them; `name` is optional) → `publish_flow`
3. `ensure_test_publishable`. If `isNew: true`, add `envLine` to `.env.local`. If `isNew: false` and env is empty, open https://app.bootstrapware.co/onboard/keys
4. `get_install_snippet`. `flowId` is real. Use the env publishable key, never invent one.
5. Embed with the real session user id, never `"user_1"` in production.
6. Live Hosted requires `authorToken` minted from your BFF session (never browser secret). Bind `workspaceKey`. Default permissions omit `reset`.

```tsx
<Onboard
  flowId="flw_..."
  publishableKey={process.env.NEXT_PUBLIC_BSW_ONBOARD_PUBLISHABLE_KEY}
  user={{ id: session.user.id, name: session.user.name }}
  workspaceKey={workspace.id}
  context={{ role: membership.role, plan: workspace.plan }}
  facts={integrationFacts}
  authorToken={authorTokenFromBff}
/>
```

`apiBaseUrl` defaults to `https://onboard.bootstrapware.co`. Live Hosted without `authorToken` is **test keys only**; production needs BFF mint + auto-renewal before expiry.

### Path C — BYO

Same MCP flow config as Path B. Implement `OnboardAdapter` on the customer backend. Progress stays on your backend; use `onEvent` for side effects. Pass `publishableKey` so the widget fetches published config.

```tsx
<Onboard
  flowId="flw_..."
  publishableKey={process.env.NEXT_PUBLIC_BSW_ONBOARD_PUBLISHABLE_KEY}
  user={currentUser}
  workspaceKey={workspace.id}
  onEvent={(event, payload) => {
    /* your analytics / CRM */
  }}
  adapter={{
    getProgress: async (input) => fetch(`/api/onboard/progress?${new URLSearchParams(input as Record<string, string>)}`).then((r) => r.json()),
    completeStep: async (input) =>
      fetch("/api/onboard/progress", { method: "POST", body: JSON.stringify({ action: "complete", ...input }) }).then((r) => r.json()),
    skipStep: async (input) =>
      fetch("/api/onboard/progress", { method: "POST", body: JSON.stringify({ action: "skip", ...input }) }).then((r) => r.json()),
    snooze: async (input) =>
      fetch("/api/onboard/progress", { method: "POST", body: JSON.stringify({ action: "snooze", ...input }) }).then((r) => r.json()),
    dismiss: async (input) =>
      fetch("/api/onboard/progress", { method: "POST", body: JSON.stringify({ action: "dismiss", ...input }) }).then((r) => r.json()),
    resume: async (input) =>
      fetch("/api/onboard/progress", { method: "POST", body: JSON.stringify({ action: "resume", ...input }) }).then((r) => r.json()),
    resetPersonal: async (input) =>
      fetch("/api/onboard/progress", { method: "POST", body: JSON.stringify({ action: "resetPersonal", ...input }) }).then((r) => r.json()),
    reportOutcome: async (input) =>
      fetch("/api/onboard/outcomes", { method: "POST", body: JSON.stringify(input) }).then((r) => r.json()),
  }}
/>
```

## MCP tools

`list_flows`, `get_flow`, `create_flow`, `update_draft` (name optional), `publish_flow`, `list_revisions`, `restore_revision`, `get_published_config`, `get_install_snippet`, `ensure_test_publishable`, `list_capabilities`.

Dashboard-only: live/secret keys, webhooks, delete, branding, billing, Hosted summaries export/preview. Test publishable: `ensure_test_publishable`.

`create_flow` / `publish_flow` return `nextSteps[]`.

## Identity and origins

Host asserts `user={{ id, name? }}` and `workspaceKey` for shared steps. Optional `authorToken` minted with a **secret** key via `POST /api/v1/author-tokens` when `requireAuthorToken` is true. Mint from your BFF using the authenticated session — never accept `authorId` from the browser blindly.

Set `allowedOrigins` on the draft (localhost + production) before live. Default `["*"]` is for first-run only.

## Mixed scope and roles

Use per-item `progressScope: "user" | "workspace"` in one flow. For different roles, publish separate flows or gate items with `eligibility`.

## Pricing

- Free: local + test keys
- BYO $9.99: live config; you store progress
- Hosted $19.99: we store dual-scope progress and integration-reported summaries
- Cancel Hosted: freeze writes, JSON export 30 days, then delete Hosted data

## Security

- Secret keys server-side only
- Publishable keys may appear in the browser
- MCP/webhooks: ids and config only — never context, facts, or progress payloads
- BFF should refresh `authorToken` before expiry; the widget does not mint tokens
