---
name: bootstrapware-answers
description: Configure and integrate Bootstrapware Answers (React package, BYO/Hosted modes, and MCP tools). Use when the user asks about @bootstrapware/answers, visitor questions with citations, publishable keys for Answers, or MCP app management.
---

# Bootstrapware Answers

## What this product does

Visitors ask a question about the page. The widget shows one complete answer with citations, or an explicit fallback when the result is `unanswered`.

**Modes**

- **Local / free:** `createLocalAdapter()`, or omit `adapter` when you also omit `publishableKey`. The sample answers hours, returns, and shipping for Northwind Outfitters. It does not call the network.
- **BYO ($9.99):** live app config. Knowledge, retrieval, the model, and visitor logs stay on the customer backend. Pass an `AnswersAdapter`, or `createByoAdapter(impl)`.
- **Hosted ($19.99):** live app config, and Bootstrapware stores approved sources, the derived index, visitor questions, optional conversations, unanswered items, and citations. Included amounts are frozen: 2,500 managed answers per workspace per month, 1 GiB of knowledge and index, 2,000 sources, 4,000 indexed chunks per app, 20 MiB per file, and 30 days of conversation history. Hitting a ceiling stops that axis. Capacity add-ons are not part of this price.

Stack discount is company-wide: 25% off the second paid product's list, 35% off the third and later. Env names only: `STRIPE_PRICE_ANSWERS_BYO` and `STRIPE_PRICE_ANSWERS_HOSTED`. Do not invent Stripe price IDs.

`@bootstrapware/answers` is not on the npm registry yet. Do not invent a version or a tarball. The imports below are the package contract. Publishable env: `NEXT_PUBLIC_BSW_ANSWERS_PUBLISHABLE_KEY`.

The React package does not call Hosted ask by itself. Passing `publishableKey` without `adapter` shows "This page is not ready to answer yet." and does not fetch.

**Never send visitor questions, file bytes, or provider keys through MCP.** MCP manages app config and public URL sources only. Never mint live or secret API keys via MCP. For a test publishable key, call `ensure_answers_test_publishable`.

## Install algorithm (do this in order)

1. Read this skill. Call `list_answers_capabilities` on Answers MCP before inventing tools.
2. If the user only needs a working UI: Path A. Do not invent keys.
3. If they need live Hosted ask: Path B.
4. If they need their own backend: Path C.
5. Wire the adapter the path requires. Omitting it does not fetch.

### Path A: local, zero keys

```tsx
import { Answers, createLocalAdapter } from "@bootstrapware/answers";
import "@bootstrapware/answers/styles.css";

<Answers adapter={createLocalAdapter()} />
```

Try these questions, in order, on one session:

1. `What is your return window?` Answered. Citation id `ane_returns`.
2. `How long do I have?` Follow-up on that same session. A new session returns unanswered.
3. `What are your hours?` Answered. Citation id `ane_hours`.
4. `Is shipping free?` Answered. Citation id `ane_shipping`.
5. `Do you price match?` Unanswered, with the sample fallback.

`onFallback` runs for unanswered results. The widget does not render an answer body for that outcome.

### Path B: Hosted

1. Confirm Cursor MCP `bootstrapware-answers` at `https://answers.bootstrapware.co/mcp` (OAuth URL-only). If disconnected, tell the human to click **Add to Cursor (OAuth)** on https://app.bootstrapware.co/answers/keys
2. `list_answers_capabilities` → `create_answers_app` → `update_answers_draft` (toggles and `allowedOrigins` for localhost and production; `name` is optional) → `publish_answers_app`
3. `ensure_answers_test_publishable`. Put `envLine` in `.env.local`. The full test publishable is returned every time. The env name is `NEXT_PUBLIC_BSW_ANSWERS_PUBLISHABLE_KEY`.
4. `get_answers_install_snippet`. `appId` is real. Use `process.env.NEXT_PUBLIC_BSW_ANSWERS_PUBLISHABLE_KEY`. Never invent a key and never put a secret in the snippet.
5. Keep the adapter in the snippet. It posts to `POST /api/v1/ask?appId=`. Omitting `adapter` does not fetch.
6. `add_answers_url_source` for each dedicated public page whose body already states the fact. Do not stop at the homepage.
7. After the index job succeeds, tell the human to retest in the dashboard playground: a product question, a follow-up that names no product, and a question the pages do not answer.

`create_answers_app` `nextSteps`: update the draft, publish, `ensure_answers_test_publishable`, `get_answers_install_snippet`, then add dedicated URL sources and retest in the playground. The React package does not call Hosted ask unless you pass an adapter. `publish_answers_app` names the test publishable tool and tells you to point the adapter at `POST /api/v1/ask`.

```tsx
import { Answers, createByoAdapter } from "@bootstrapware/answers";
import "@bootstrapware/answers/styles.css";

<Answers
  appId="ana_demo"
  publishableKey={process.env.NEXT_PUBLIC_BSW_ANSWERS_PUBLISHABLE_KEY}
  adapter={createByoAdapter({
    ask: async (input) => {
      const response = await fetch("https://answers.bootstrapware.co/api/v1/ask?appId=ana_demo", {
        method: "POST",
        headers: {
          Authorization: `Bearer ${process.env.NEXT_PUBLIC_BSW_ANSWERS_PUBLISHABLE_KEY}`,
          "Content-Type": "application/json",
        },
        body: JSON.stringify(input),
      });
      const json = await response.json();
      if (!response.ok) throw new Error(json?.error?.message ?? "Could not ask.");
      return json.data;
    },
  })}
/>
```

`get_answers_install_snippet` substitutes the published `ana_` id. Optional API host in the snippet defaults to `https://answers.bootstrapware.co`.

The ask body is `{ sessionId, question, pageContext }`. `pageContext` is `{ url, title }` only. The widget does not send the DOM. Success is `{ data }` with `outcome` `answered` or `unanswered`. An answered result includes `answer` and citations whose ids came from retrieval. An unanswered result includes `fallback` and no answer text. There is no public confidence percentage. The response is complete. It does not stream.

Live publishable config and live Hosted writes need a paid plan. Test keys work unpaid. Live Hosted writes need Hosted ($19.99). Live config needs BYO ($9.99) or Hosted ($19.99).

Public URL sources are `add_answers_url_source` with `kind` `website`, `sitemap`, or `url`. `reindex_answers_app` takes that same URL, or omits `url` to reindex the app. Neither tool accepts file bytes or a question.

### Path C: BYO

Same MCP app config as Path B. Implement `AnswersAdapter` on the customer backend. `createByoAdapter(impl)` returns that adapter. Knowledge, retrieval, the model, and visitor logs never go to Bootstrapware.

`ask` receives `{ sessionId, question, pageContext }` and returns `{ sessionId, answerId, outcome, answer?, citations?, fallback? }`. `outcome` is `answered` or `unanswered`. Optional `feedback({ sessionId, answerId, helpful })`.

```tsx
<Answers
  appId="ana_demo"
  adapter={createByoAdapter({
    ask: async (input) => {
      const response = await fetch("/api/answers/ask", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(input),
      });
      if (!response.ok) throw new Error("Could not ask.");
      return response.json();
    },
  })}
/>
```

The reference backend is `examples/answers-nextjs`. It reads markdown knowledge on the server, ranks chunks, and grounds the answer on retrieved ids. `OPENAI_API_KEY` stays in server env. Without that key, the route quotes a retrieved sentence and does not call a model.

Citations must be ids you retrieved. The widget links a URL only when that URL is on the returned citation. Other URLs stay text. HTML in the answer stays text.

## Hosted MCP (this plugin)

Endpoint: `https://answers.bootstrapware.co/mcp`

Server id: `bootstrapware-answers`

**Preferred:** OAuth Connect: URL-only config, Connect in Cursor, sign in at https://app.bootstrapware.co. Scope `answers:mcp`. **Add to Cursor (OAuth)** on https://app.bootstrapware.co/answers/keys

**Fallback:** test secret as `Authorization: Bearer` (mint at https://app.bootstrapware.co/answers/keys). Live and secret keys stay on the dashboard.

### Tools

App config and public URL sources only: `list_answers_apps`, `get_answers_app`, `create_answers_app`, `update_answers_draft` (name optional; does not publish), `publish_answers_app`, `get_answers_published_config`, `get_answers_install_snippet`, `ensure_answers_test_publishable`, `list_answers_capabilities`, `add_answers_url_source` (public `http` or `https` URL only), `reindex_answers_app` (that URL, or the app when `url` is omitted).

`update_answers_draft` accepts `assistantLabel`, `greeting`, `theme`, `presentation`, `fallbackMessage`, `allowedOrigins`, `retainHistory`, and `publicAnsweringEnabled`. It rejects questions, answers, file bytes, and provider keys. `allowedOrigins` `["*"]` is the CORS default for the ask route. It is not content authorization. `publicAnsweringEnabled: false` pauses new asks. Sources and history stay.

Dashboard-only: live and secret key mint, webhooks, app delete, billing, file upload, manual sources, conversations, unanswered text, provider secrets, export, purge. Test publishable: `ensure_answers_test_publishable`.

`create_answers_app` and `publish_answers_app` return `nextSteps[]`. Call `list_answers_capabilities` before using any other tool name. Capabilities include `sourceAdvice`: index dedicated public pages, name the product in the priced sentence, and retest follow-ups in the playground.

## Publish pages that can be quoted

Hosted quotes the body after it drops navigation chrome. A homepage of product blurbs is a weak first source.

1. Add a dedicated public URL per fact (`/pricing`, hours, returns).
2. Put the product name and the price in the same sentence.
3. Put each product's comparison on that product's page.
4. Unanswered is success when the pages do not have the fact. Add a URL or a dashboard manual FAQ. Do not send the question through MCP.

Publishing stores hosted app config. There is no `GET /api/v1/config/:appId` route. `POST /api/v1/ask` reads the published revision.

## Props and session

- `appId`: published app id (`ana_`).
- `publishableKey`: browser publishable key. Unused for fetch unless your adapter uses it.
- `adapter`: `ask`, and optional `feedback`. Required for Hosted and BYO. Omitting it does not fetch.
- `presentation`: `launcher` (default), `section`, `modal`, or `window`.
- `pageContext`: `{ url, title }`. Omit it and the widget sends `location.href` and `document.title`.
- `onFallback`: called with the sanitized unanswered result.

The widget stores `ans_` plus 24 hex in `sessionStorage` under `bsw_an_session:<appId>` (`local` when `appId` is omitted). It does not write a cookie. Questions are limited to 1,000 Unicode code points. A longer draft stays in the composer and is not sent.

`GET https://answers.bootstrapware.co/embed/v1.js` is the same widget. `BootstrapwareAnswers.mount(el, props)` returns an unmount function. Without an adapter, the script does not fetch.

## Knowledge and provider

Hosted knowledge kinds are website root, sitemap, single public URL, PDF, DOCX, TXT, Markdown, and manual FAQ. MCP can add a public URL and reindex it. Files, manual text, conversations, and unanswered question text stay on the dashboard.

Every Hosted source is visitor-readable. Do not upload private runbooks or secrets. Crawl is public `http` and `https` only. A failed crawl leaves the last good index in place. A page that is only navigation chrome is rejected and does not become a citation.

Citations are ids from the retrieved revision. Unsupported and plausible-false questions return `unanswered` and the configured fallback. There is no model-memory fallback.

Hosted AI is managed OpenAI, or one Hosted BYOK OpenAI key. The provider secret stays on `GET`, `PUT`, and `DELETE /api/v1/apps/:id/provider` with a secret key. MCP, the browser package, logs, and publishable config do not receive the key. Anthropic and Gemini are not in this version.

## Pricing

- Free: local mode and test keys
- BYO $9.99: live config; you store knowledge and visitor logs
- Hosted $19.99: we store sources, index, questions, optional conversations, unanswered items, and citations inside the frozen envelope above
- Cancel Hosted: writes freeze immediately. Dashboard JSON export for 30 days, then Hosted sources, index, conversations, and unanswered items are deleted
- Hosted to BYO keeps the rows and does not start the 30-day clock

## Security

- Secret keys server-side only. Do not put `bsw_live_sec_` or `bsw_test_sec_` in client code, `NEXT_PUBLIC_*`, logs, or snippets.
- Do not send visitor questions, file bytes, or provider keys to MCP or webhooks.
- Publishable keys and `allowedOrigins` identify the workspace and apply CORS. They do not stop a script that replays the publishable key. Hosted ask still checks the key, the body size, the burst limits, and the monthly allowance before retrieval or a model call.
- Visitor question text and indexed documents are untrusted. Render the safe markdown subset only.
- Local and BYO question text stays in the browser or on the customer backend.
