## flipagent

> ONE API for online reselling. The hosted service at `api.flipagent.dev`

# flipagent

ONE API for online reselling. The hosted service at `api.flipagent.dev`
gives AI agents and apps a unified surface for the full reseller cycle
(discovery → evaluation → buying → listing → fulfillment → finance)
across marketplaces. Today: eBay (REST mirror + scrape fallback). Soon:
Amazon, Mercari, Poshmark.

The whole API server is OSS (recall.ai-style: open backend, hosted
operations as the moat). Page rendering for `EBAY_*_SOURCE=scrape` is
delegated to a managed Web Scraper API (today: Oxylabs) — we POST a
URL and they return rendered HTML on their infrastructure, under their
upstream-marketplace ToS. flipagent's own code path is a normal HTTPS
client; it does not implement UA rotation, browser fingerprinting, or
any other vendor-side concern.

## Workspaces

| Path | Name | License | Role |
|---|---|---|---|
| `packages/types` | `@flipagent/types` | MIT | TypeBox schemas for flipagent's own `/v1/*` — `evaluate`, `discover`, `ship` (intelligence layer) plus errors, tier, billing, keys, takedown, health |
| `packages/types/ebay` | `@flipagent/types/ebay` | MIT | TypeBox schemas mirroring eBay REST shapes — `/buy` (Browse + Marketplace Insights) and `/sell` (Inventory, Fulfillment) subpaths |
| `packages/ebay-scraper` | `@flipagent/ebay-scraper` | MIT | eBay HTML parsers + plain-HTTP fetcher (BYO proxy) |
| `packages/sdk` | `@flipagent/sdk` | MIT | Typed client. Marketplace passthrough namespaces (`listings`, `sold`, `orders`, `inventory`, `fulfillment`, `finance`, `markets`) plus flipagent intelligence (`research`, `match`, `evaluate`, `discover`, `ship`, `draft`, `reprice`, `expenses`) and ops (`webhooks`, `capabilities`). |
| `packages/mcp` | `flipagent-mcp` | MIT | MCP server — exposes eBay tools + deal-finding tools to Claude Desktop / Cursor / Cline. |
| `packages/cli` | `flipagent-cli` | MIT | One-command MCP setup. Detects Claude Desktop / Cursor and writes the `flipagent` server entry. `npx -y flipagent-cli init --mcp --keys`. |
| `packages/api` | `@flipagent/api` | FSL-1.1-ALv2 (private — not published, source on GitHub; converts to Apache 2.0 two years after each release) | Hono backend: unified API surface (eBay-compat + `/v1/*`), scraping, scoring, auth, billing. |
| `apps/docs` | `@flipagent/docs` | proprietary (All Rights Reserved) | flipagent.dev marketing + dashboard site (Astro static). Source visible for transparency; redistribution / rebrand not permitted. |

## Dependency direction

```
   @flipagent/types ──┐
                      ├──►  @flipagent/sdk  ──►  flipagent-mcp  (npm)
                      │            │
                      │            │ HTTPS
                      │            ▼
                      │     api.flipagent.dev
                      │            │  (= @flipagent/api)
                      │            │
                      ├──►  @flipagent/api  ──►  Postgres
                      │     (Hono backend)       Oxylabs Web Scraper API
   @flipagent/ebay-scraper ─────►  │              eBay / Amazon / Mercari (future)
                                   │
                                   └──►  services/{scoring,quant,forwarder} (server-side math)
```

`flipagent-mcp` calls flipagent's hosted API through `@flipagent/sdk`.
Math (median, margin, scoring, recipes) runs **server-side** in
`packages/api/src/services/scoring/` so all SDK clients in any language
get the same scoring without re-implementing it.

## Structural rules

- **Marketplace-agnostic surface.** Endpoints live under `/v1/*` in
  three layers. The mirror layer mirrors eBay's REST path shape
  verbatim (`/v1/buy/*`, `/v1/sell/*`, `/v1/commerce/*`,
  `/v1/post-order/*`) so agents can read eBay docs and call our routes
  one-to-one.
  - Marketplace mirror — Buy:
    sourcing reads (`/v1/buy/browse/item_summary/search`,
    `/v1/buy/browse/item/{itemId}`,
    `/v1/buy/browse/item/get_items`,
    `/v1/buy/browse/item/get_items_by_item_group`),
    sold comparables (`/v1/buy/marketplace_insights/item_sales/search`),
    buy queue (`/v1/buy/order/*` — single surface, REST + bridge transports),
    bulk ops (`/v1/buy/feed/*`, `/v1/buy/deal/*`),
    buyer-side bidding (`/v1/buy/offer/*`).
  - Marketplace mirror — Sell:
    listing CRUD (`/v1/sell/inventory/*`),
    order ops (`/v1/sell/fulfillment/*`),
    payouts (`/v1/sell/finances/*`),
    selling policies (`/v1/sell/account/{fulfillment,payment,return}_policy`),
    seller marketing (`/v1/sell/marketing/*`),
    Best Offer outbound (`/v1/sell/negotiation/*`),
    seller perf (`/v1/sell/analytics/*`, `/v1/sell/compliance/*`,
    `/v1/sell/recommendation/*`),
    bulk ops (`/v1/sell/feed/*`),
    shipping labels (`/v1/sell/logistics/*`),
    eBay Stores (`/v1/sell/stores/*`),
    sell metadata (`/v1/sell/metadata/*`).
  - Marketplace mirror — Commerce (cross-cutting):
    taxonomy (`/v1/commerce/taxonomy/*`),
    catalog (`/v1/commerce/catalog/*`),
    connected user (`/v1/commerce/identity/*`),
    translation (`/v1/commerce/translation/*`).
  - Marketplace mirror — Post-order:
    returns/cases/cancellations/inquiries/issues (`/v1/post-order/*`).
  - Trading API XML wrappers exposed as JSON (no REST equivalent on
    eBay): `/v1/messages`, `/v1/best-offer` (inbound Best Offer
    respond), `/v1/feedback`.
  - flipagent intelligence — `/v1/research`, `/v1/match`,
    `/v1/evaluate`, `/v1/discover`, `/v1/ship`, `/v1/draft`,
    `/v1/reprice`, `/v1/expenses`, `/v1/traces`, `/v1/trends`.
  - Forwarder ops (provider-namespaced; used in both buy + sell flows):
    `/v1/forwarder/{provider}/*`.
  - Account/ops — `/v1/{keys,billing,connect,me,takedown,capabilities,health}`.
  - Agent plumbing — `/v1/bridge` (extension wire protocol),
    `/v1/browser` (browser-agent integration), `/v1/notifications`
    (eBay Trading inbound webhook), `/v1/webhooks` (outbound dispatch
    to user-registered endpoints), `/v1/queue`, `/v1/watchlists`.
  `/v1/buy/order/*` is the single Buy Order surface with two
  first-class transports — `rest` and `bridge` — selected by
  `selectTransport` per the capability matrix + `?transport=`
  override + `EBAY_ORDER_API_APPROVED`. New marketplaces (Amazon,
  Mercari, …) reuse the mirror paths via a `marketplace` parameter
  rather than path prefixes. Internally, the REST passthrough
  rewrites our `/v1/<group>/<resource>/...` to eBay's
  `/<group>/<resource>/v1/...` shape (the per-API version segment
  shifts from prefix to between resource and operation) — see
  PATH_MAP in `packages/api/src/services/ebay/rest/client.ts`.
- **Provider / resource / route layering.** The eBay provider lives at
  `packages/api/src/services/ebay/{rest,scrape,bridge,trading}/` —
  one folder per transport, all eBay-specific code and only that.
  Resource services at `services/<resource>/*` (listings, match,
  inventory, …) are marketplace-agnostic business logic; they pick a
  transport via `services/shared/transport.ts` (`selectTransport` +
  `RESOURCE_TRANSPORTS` capability matrix) and dispatch into the
  provider. Routes at `routes/v1/*` validate input, call the
  resource service, render headers via `renderResultHeaders`. Every
  resource service returns the shared `FlipagentResult<T>` envelope
  from `services/shared/result.ts`. Cross-cutting cache wrapping is
  `services/shared/with-cache.ts`. Future Amazon / Mercari adapters
  drop in as `services/amazon/`, `services/mercari/` siblings of
  `services/ebay/`, with their own capability matrices.
- **OSS code never imports `apps/docs/*`.** Docs site is closed; never
  reach into it from packages.
- **Scraping is OSS, the vendor creds are env.** `@flipagent/ebay-scraper`
  ships pure parsers. The fetch path (vendor dispatcher) lives in
  `packages/api/src/services/ebay/scrape/` and is OSS too; the shared
  response cache primitives sit in `services/shared/cache.ts`. The
  managed Web Scraper API takes a URL and returns rendered HTML —
  whatever rendering, IP routing, or JS execution the vendor performs
  is on their side, under their own upstream-marketplace ToS. flipagent
  does not ship a UA pool, browser fingerprinting, or any equivalent
  vendor-side logic of its own. The vendor is selected via
  `SCRAPER_API_VENDOR` (today only `oxylabs` is wired) with credentials
  in `SCRAPER_API_USERNAME` / `SCRAPER_API_PASSWORD`. Adding a vendor =
  drop an adapter at
  `packages/api/src/services/ebay/scrape/scraper-api/<vendor>.ts`
  plus a case in the dispatcher.
- **SDK is a hand-rolled thin client.** `createFlipagentClient` returns
  one client with ergonomic namespaces that internally call the
  verbose `/v1/*` paths. Three groups:
  marketplace passthrough (`listings`, `sold`, `buy.order`, `inventory`,
  `fulfillment`, `finance`, `markets`, `forwarder`), flipagent
  intelligence (`research`, `match`, `evaluate`, `discover`, `ship`,
  `draft`, `reprice`, `expenses`), and ops (`webhooks`, `capabilities`).
  Namespace names don't map 1:1 to URL segments — e.g. `client.listings`
  hits `/v1/buy/browse/*`, `client.sold` hits
  `/v1/buy/marketplace_insights/item_sales/search`, `client.markets`
  splits into `taxonomy` (→ `/v1/commerce/taxonomy/*`) and `policies`
  (→ `/v1/sell/account/{fulfillment,payment,return}_policy`). No
  vendored eBay client in the user-facing path — the SDK speaks HTTPS
  to `api.flipagent.dev` directly. New endpoints get a typed namespace
  on the SDK; the underlying `client.http.{get,post,...}` is the escape
  hatch.
- **Cents in code, dollar strings on the wire.** Internal Listing /
  margin / scoring use cents-denominated integers. eBay's API uses
  string dollars on the wire; convert at the API boundary
  (`packages/api/src/proxy/scrape.ts`).
- **TypeBox for schemas.** Hono routes validate request bodies via
  `Value.Errors(Schema, body)`. Zod is banned in agent/tool surfaces.
- **Auth is X-API-Key or Authorization: Bearer.** Plaintext shown once
  at creation; only the sha256 hash persists. Tier limits enforced per
  calendar month (UTC) via `usage_events` count.

## Environment

`packages/api/.env.example` is the source of truth. Required:
`DATABASE_URL`. Recommended for any meaningful scrape volume:
`SCRAPER_API_VENDOR` (default `oxylabs`) plus `SCRAPER_API_USERNAME` /
`SCRAPER_API_PASSWORD`. Without those the scrape paths still work for
small-volume use but datacenter HTTP responses degrade quickly under
sustained load. Stripe billing is
opt-in: set `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`,
`STRIPE_PRICE_HOBBY`, `STRIPE_PRICE_PRO` together or not at all —
`/v1/billing/*` returns 503 when any are missing.

eBay OAuth passthrough is opt-in too: set `EBAY_CLIENT_ID`,
`EBAY_CLIENT_SECRET`, `EBAY_RU_NAME` together or not at all —
`/v1/connect/ebay/*`, every `/sell/*`, and `/commerce/*` route returns
503 when any are missing. Default `EBAY_BASE_URL=https://api.ebay.com`
(swap to sandbox by setting `EBAY_BASE_URL` + `EBAY_AUTH_URL` to
`*.sandbox.ebay.com`).

`/v1/buy/order/*` (Buy Order API mirror) is single surface with two
**first-class** transports — `rest` and `bridge`. Both produce the
same eBay-shape `CheckoutSession` / `EbayPurchaseOrder` response;
neither is a "fallback" for the other. `selectTransport` (in
`services/shared/transport.ts`) picks one given the capability
matrix + per-call `?transport=` override + env flags
(`EBAY_ORDER_API_APPROVED`). REST requires the env flag + the api
key's eBay OAuth binding; bridge requires a paired Chrome
extension. The 2-stage flow (`initiate` → `place_order`) is fully
implemented in both transports. Multi-stage update endpoints
(`shipping_address`, `payment_instrument`, `coupon`) only work in
REST transport — bridge uses the buyer's stored eBay defaults so
those return 412 with a clear pointer to switch transport.

**Bridge-driven non-buy ops have their own surface, not a generic
`/v1/orders/*` queue.** Each source the bridge handles maps to a
typed public surface:

  - `/v1/buy/order/*` — eBay buy (REST + bridge transports, eBay shape)
  - `/v1/forwarder/{provider}/*` — package forwarder ops (Planet
    Express today; used in both buy + sell flows so it sits at top
    level, not under `/buy/` or `/sell/`)
  - `/v1/browser/*` — synchronous DOM primitives (browser_op)
  - control / extension reload — internal admin only

`/v1/bridge/*` and `/v1/browser/*` are NOT the same thing — different
layers serving different audiences:
  - `/v1/bridge/*` = the wire protocol the Chrome extension uses to
    talk to flipagent (token issuance + longpoll + result reporting +
    login-status). Audience: the extension itself.
  - `/v1/browser/*`, `/v1/buy/order/*`, `/v1/forwarder/*`, etc. =
    user-facing surfaces that internally queue work via the bridge.
    Audience: agents / SDK callers.

The shared bridge queue infra lives in `services/orders/queue.ts`
(name predates the split). Each public surface calls `createOrder`
with its own `source` value; the bridge route maps source → task name
via `services/ebay/bridge/tasks.ts`. Future `services/orders/queue.ts`
rename to `services/bridge-jobs/` is queued as a separate cleanup.

## Code conventions

- TypeScript strict, `"type": "module"`, `Node16` moduleResolution,
  ES2022 target.
- Tab indent, width 3, line width 120 (Biome 2.3.x).
- No `any` unless commented with reason.
- Top-level `import` only; no inline dynamic imports in source.
- Each package: `src/`, optional `test/`, `package.json`,
  `tsconfig.build.json`, `dist/` (gitignored).

## Commands

- `npm install` — bootstrap all workspaces.
- `npm run typecheck` — full-repo `tsc --noEmit`.
- `npm run check` — biome + typecheck.
- `npm run build` — composite build of types → ebay-scraper → sdk →
  api → mcp → cli → docs (in dependency order).
- `npm test` — vitest in each workspace that has tests.
- `docker compose up -d postgres` — local Postgres on `localhost:55432`.
- `cd packages/api && npm run db:migrate` — apply drizzle migrations.

## ToS hygiene (eBay)

- Never redistribute raw listing content (titles, descriptions, photos,
  seller details) divorced from `itemWebUrl`. Cached responses always
  carry the original `ebay.com/itm/...` URL.
- Cache TTL is short (60 min active, 12h sold, 4h detail). The cache
  is anti-thundering-herd, not archival.
- `/v1/takedown` accepts seller opt-out. Approved takedowns flush the
  cache and blocklist the itemId. Doubles as our GDPR Art. 17 / CCPA
  delete-request channel — same pipe, three regulatory regimes.
- Outbound scrape traffic to ebay.com is delegated to the managed
  scraper vendor (`SCRAPER_API_VENDOR`), not issued from flipagent's
  own IPs — so we don't hammer ebay.com directly.

## Deploy

- `packages/api` → Azure Container Apps via `infra/azure/` Terraform.
  Container Registry pushes from `az acr build`; a system-assigned
  identity has `AcrPull`. Postgres Flexible Server is reachable via the
  "Allow Azure services" firewall rule. `MIGRATE_ON_BOOT=1` is set on
  the Container App env so drizzle migrations run before the api starts
  (idempotent; revisit if `min_replicas` ever exceeds 1).
- `apps/docs` → Cloudflare Pages. Static `dist/`.
- OSS packages → npm publish via Changesets. Workflow: `npx changeset`
  on a PR to declare what changed and at what bump (patch/minor/major)
  per package. On merge to `main`, `.github/workflows/release.yml`
  either opens a "Version Packages" PR (if changesets are pending) or
  publishes the bumped packages with npm provenance (if a previous
  Version PR was just merged). `NPM_TOKEN` secret must be set; private
  packages (`@flipagent/api`, `@flipagent/docs`) are skipped via
  `"private": true`.

## When extending

- New eBay endpoint (any transport) → 1) add a one-line entry to
  `RESOURCE_TRANSPORTS` in `services/shared/transport.ts` declaring
  which transports the resource supports + their auth/task/call
  needs. 2) add or extend a resource service under
  `services/<resource>/*` that calls `selectTransport(...)` and
  dispatches into the eBay provider folders below. 3) add a thin
  route in `routes/v1/<resource>.ts`. 4) Schema in
  `packages/types/src/ebay/{buy,sell}.ts`, SDK namespace, MCP tool
  as needed.
- **eBay REST passthrough (PATH_MAP).** When the resource is a thin
  proxy to `api.ebay.com` REST, append a PATH_MAP entry in
  `services/ebay/rest/client.ts` (one line, regex → eBay path
  prefix) and have the route mount `ebayPassthroughUser` (user
  OAuth) or `ebayPassthroughApp` (app credential). Cached
  read-only resources go through the `cacheFirst` middleware from
  `middleware/cache-first.ts`.
- **eBay Trading API call (XML/SOAP).** Add a service in
  `services/ebay/trading/` (use `client.ts` helpers `tradingCall`,
  `parseTrading`, `escapeXml`). Add a v1 route that wraps it as
  JSON; wrap the handler in `withTradingAuth(...)` from
  `middleware/with-trading-auth.ts` — it resolves user OAuth,
  surfaces 401 `ebay_account_not_connected`, maps `TradingApiError`
  uniformly. See `routes/v1/{messages,best-offer,feedback}.ts`.
- **eBay bridge task.** Add a constant to `BRIDGE_TASKS` in
  `services/ebay/bridge/tasks.ts`, declare bridge capability in
  `RESOURCE_TRANSPORTS`, and have the resource service queue the
  task through the existing bridge queue (`services/orders/queue`).
  The Chrome extension picks it up via `/v1/bridge/poll`.
- **Service-result envelope.** Every resource service returns
  `FlipagentResult<T> = { body, source, fromCache, cachedAt? }` from
  `services/shared/result.ts`. `source` is one of
  `"rest" | "scrape" | "bridge" | "trading" | "llm"` — the data
  origin, never `"cache:..."` (cache hits flip `fromCache`).
  Routes call `renderResultHeaders(c, result)` from
  `services/shared/headers.ts` to set `X-Flipagent-Source` +
  `X-Flipagent-From-Cache` + `X-Flipagent-Cached-At`. Wrap upstream
  calls in `withCache(args, fetcher)` from
  `services/shared/with-cache.ts` — one canonical cache-or-fetch
  flow with built-in upstream timeout. Pure-passthrough cached
  reads (taxonomy, catalog, sell-metadata) use the `cacheFirst()`
  middleware from `middleware/cache-first.ts`, which itself
  delegates to `withCache` so internals stay aligned.
- New flipagent-specific endpoint → put it under `/v1/`. Schema in
  `packages/types/src/` — `research.ts` / `evaluate.ts` / `discover.ts`
  / `ship.ts` / `draft.ts` / `reprice.ts` / `expenses.ts` for the
  intelligence layer, `flipagent.ts` for account/ops, or a new file
  matching the route namespace.
- New scoring algorithm → goes to `packages/api/src/services/scoring/`
  (or `quant/` for low-level stats, `forwarder/` for shipping rates).
  Pure functions, no I/O, cents-denominated. Add a vitest in
  `packages/api/test/services/`.
- New scraper helper → goes to `@flipagent/ebay-scraper` only if it's
  pure parsing. Anything that talks to a proxy or DB belongs in
  `packages/api/src/proxy/`.
- New marketplace adapter (Amazon, Mercari, etc.) → goes in
  `packages/api/src/adapters/<marketplace>/`. Register routes under
  the unified `/listings/*`, `/orders/*` etc. surface (not
  marketplace-specific paths).

---
> Source: [flipagent/flipagent](https://github.com/flipagent/flipagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
