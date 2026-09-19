---
name: bootstrapware-onboard
description: Configure and integrate Bootstrapware Onboard (React package, BYO/Hosted modes, MCP tools, install algorithm). Use when the user asks about @bootstrapware/onboard, first-run setup checklists, Hosted Onboard, or MCP flow management.
---

# Bootstrapware Onboard

Embeddable first-run setup checklist for SaaS: required and optional steps, dependencies, eligibility, fact reconciliation, dismiss, and snooze. Userflow and similar products also offer MCP — Onboard is not unique in that regard.

**Never send host context, flags, facts, or progress payloads through MCP.** Never mint API keys via MCP. Never invent `user.id` for production.

## Install algorithm (do this in order)

1. Read this skill. Call `list_capabilities` on Onboard MCP before inventing tools.
2. If the user only needs a working UI: Path A. Do not invent keys.
3. If they need live Hosted or BYO config: Path B or C.
4. Wire the **real** session id and `workspaceKey` from the host app.
5. If no publishable key is in env, **stop and ask the human**.

### Path A — local, zero keys

```bash
pnpm add @bootstrapware/onboard
```

```tsx
import { Onboard, createLocalAdapter } from "@bootstrapware/onboard";
import "@bootstrapware/onboard/styles.css";
import { SAAS_FIRST_RUN_ITEMS } from "@bootstrapware/sdk";

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
2. `list_capabilities` → `create_flow` (SaaS first-run template) → `update_draft` (items, eligibility, `allowedOrigins`) → `publish_flow`
3. `get_install_snippet`. `flowId` is real. The key is a placeholder (`YOUR_PUBLISHABLE_KEY`).
4. If env has no `bsw_test_pub_` / `bsw_live_pub_` value, **stop**. Ask the human to mint a test publishable key at https://app.bootstrapware.co/onboard/keys
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

`list_flows`, `get_flow`, `create_flow`, `update_draft`, `publish_flow`, `list_revisions`, `restore_revision`, `get_published_config`, `get_install_snippet`, `list_capabilities`.

Dashboard-only: keys, webhooks, delete, branding, billing, Hosted summaries export/preview.

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
