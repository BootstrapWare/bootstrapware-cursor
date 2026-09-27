---
name: bootstrapware-rfq
description: Configure and integrate Bootstrapware RFQ (React package, BYO/Hosted modes, and MCP tools). Use when the user asks about @bootstrapware/rfq, RfqRequester, RfqSupplier, RfqComparison, publishable keys for RFQ, or MCP app management.
---

# Bootstrapware RFQ

## What this product does

Embeddable buyer–supplier requests for quote. Surfaces: `RfqRequester`, `RfqSupplier`, and `RfqComparison`. `Rfq` takes an explicit `surface`. Those three omit it. `rfqDraftKey(actorId, rfqId, field)` is `bsw-rfq:${actorId}:${rfqId}:${field}` in `sessionStorage`.

**Modes**

- **Local / free:** `createLocalAdapter({ storageKey: "demo-rfq" })`.
- **BYO ($9.99):** you store RFQ records via `RfqAdapter` or `createByoAdapter`. Bootstrapware hosts app config only.
- **Hosted ($19.99):** Bootstrapware stores RFQ records. Attachment bytes stay on the host. Selection is intent only.

`@bootstrapware/rfq` is not published to npm and is not marketplace-listed. Do not npm publish it. Do not invent Stripe price IDs. Env names are `STRIPE_PRICE_RFQ_BYO` and `STRIPE_PRICE_RFQ_HOSTED`.

**Never send RFQ titles, quotes, prices, supplier lists, or file bytes through MCP.** MCP manages app configuration only. Never mint live or secret API keys via MCP. For a test publishable key, call `ensure_rfq_test_publishable`.

## Install

```bash
pnpm add @bootstrapware/rfq
```

```tsx
import { RfqRequester, createLocalAdapter } from "@bootstrapware/rfq";
import "@bootstrapware/rfq/styles.css";
```

Peer dependencies: `react` and `react-dom` (>= 18).

## Identity

Bootstrapware does **not** authenticate RFQ visitors. Your app asserts opaque `actor={{ id, permissions }}`. Use the real session id (`session.user.id`).

`scope` (`appId`, `tenantKey`) is a widget prop. Those strings are not proof of access.

Optional `authorToken` is minted on your server with an RFQ **secret** key (`POST https://rfq.bootstrapware.co/api/v1/author-tokens`). Prefix `bsw_rfqauth_v1.`. Never mint it in the browser or via MCP. Pass `renewAuthorToken` so the widget can refresh it. The secret stays in server env (`BSW_RFQ_SECRET`), never `NEXT_PUBLIC_*`.

## Local

```tsx
<RfqRequester
  scope={{ appId: "rqa_local", tenantKey: workspace.id }}
  actor={{ id: session.user.id, permissions: ["buyer_read", "buyer_edit"] }}
  adapter={createLocalAdapter({ storageKey: "demo-rfq" })}
/>
```

`RfqSupplier` and `RfqComparison` use the same props with `surface` omitted. Omit `theme` and the shell stays light: no `data-theme`, and no `prefers-color-scheme`. Saved JSON that cannot be read shows: `Saved RFQs could not be read. Showing an empty in-memory RFQ.`

## Hosted

```tsx
import { RfqComparison, RfqRequester, RfqSupplier, createHostedAdapter } from "@bootstrapware/rfq";

const adapter = createHostedAdapter({
  appId: "rqa_...",
  publishableKey: process.env.NEXT_PUBLIC_BSW_RFQ_PUBLISHABLE_KEY!,
  scope: { tenantKey: workspace.id },
  authorToken,
});
```

`RfqSupplier` and `RfqComparison` take the same scope, actor, and adapter. `apiBaseUrl` defaults to `https://rfq.bootstrapware.co`. That host name is not a DNS record.

Path B: `list_rfq_capabilities` → `create_rfq_app` → `update_rfq_draft` (config and `allowedOrigins`; `name` optional) → `publish_rfq_app` → `ensure_rfq_test_publishable` (put `envLine` in `.env.local`; the full test publishable is returned every time) → `get_rfq_install_snippet`. Embed `RfqRequester`, `RfqSupplier`, or `RfqComparison` with the real session actor id.

`create_rfq_app` `nextSteps`: update the draft, publish, `ensure_rfq_test_publishable`, then `get_rfq_install_snippet`. `publish_rfq_app` names the same two tools and the three surfaces.

`update_rfq_draft` `config` accepts only `preset`, `currencies`, `units`, `questions`, `labels`, `theme`, `comparisonColumns`, `requireAuthorToken`, and `allowedOrigins`. Questions are schema, not answers. Send `theme: null` to clear theme.

## Hosted MCP (this plugin)

Endpoint: `https://rfq.bootstrapware.co/mcp`

Server id: `bootstrapware-rfq`

**Preferred:** OAuth Connect: URL-only config, Connect in Cursor, sign in at https://app.bootstrapware.co. Scope `rfq:mcp`.

**Fallback:** secret key as `Authorization: Bearer` (mint at https://app.bootstrapware.co/rfq/keys).

### Tools

App config only: `list_rfq_apps`, `get_rfq_app`, `create_rfq_app`, `update_rfq_draft` (name optional), `publish_rfq_app`, `get_rfq_published_config`, `get_rfq_install_snippet`, `ensure_rfq_test_publishable`, `list_rfq_capabilities`. No RFQ list, quote submit, prices, supplier lists, or file bytes.

Dashboard-only: live and secret key mint, webhooks, app delete, branding, billing, Hosted export, purge, config revision restore. Test publishable: `ensure_rfq_test_publishable`.

`create_rfq_app` and `publish_rfq_app` return `nextSteps[]`. Call `list_rfq_capabilities` before using any other tool name.

## Pricing

- Free: local + test keys
- BYO $9.99: live config; you store records
- Hosted $19.99: we store records, 1 GiB per environment, attachment bytes excluded
- Cancel Hosted: freeze writes, export 30 days, then delete Hosted data
- Hosted → BYO keeps the rows and does not start the 30-day clock
- Selection is intent only

## Security

- Secret keys server-side only
- Publishable keys and `allowedOrigins` do not prove the end user may read or write
- Do not invent Stripe price IDs
