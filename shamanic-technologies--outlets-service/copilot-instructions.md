## outlets-service

> | Tier | Prefix | Auth | Example |

# outlets-service-v1

## Route Structure

| Tier | Prefix | Auth | Example |
|------|--------|------|---------|
| Public | `/` | None | `GET /health`, `GET /openapi.json` |
| Internal | `/internal/*` | `x-api-key` | `POST /internal/enrich` |
| Org-scoped | `/orgs/*` | `x-api-key` + `x-org-id` | `GET /orgs/outlets`, `POST /orgs/outlets/discover` |

## Auth Model

Two middlewares, applied in `app.ts`:
1. **`apiKeyAuth`** — validates `x-api-key` against `OUTLETS_SERVICE_API_KEY`. Crashes at startup if env var missing.
2. **`requireOrgId`** — requires `x-org-id`, parses all other identity headers as optional into `req.orgContext` (`OrgContext` type).

Middleware order in `app.ts`:
```
Public routes (no middleware)
  └─ apiKeyAuth
       ├─ /internal/* (API key only)
       └─ requireOrgId
            └─ /orgs/* (API key + org context)
```

## Tenant Isolation

Every `/orgs/*` SQL query MUST include `WHERE org_id = $N` — no exceptions. This prevents cross-org data leaks.

## OrgContext Type

```typescript
interface OrgContext {
  orgId: string;        // required (from x-org-id)
  userId?: string;      // optional (from x-user-id)
  runId?: string;       // optional (from x-run-id)
  campaignId?: string;  // optional (from x-campaign-id)
  brandIds: string[];   // optional (from x-brand-id, comma-split)
  featureSlug?: string; // optional (from x-feature-slug)
  workflowSlug?: string;// optional (from x-workflow-slug)
}
```

## Header Forwarding

`buildServiceHeaders(apiKey, ctx)` in `src/services/headers.ts` — includes `x-api-key` and `x-org-id` unconditionally, all others only when present.

## Cross-service enrichment: fail-loud vs best-effort

Two classes of cross-service read enrichment, and they have OPPOSITE failure modes — don't apply one rule to both:

- **Core data → fail-loud.** Enrichment the endpoint's purpose depends on (e.g. journalist outreach status from journalists-service on `GET /orgs/outlets`, which drives `byOutreachStatus`). On failure, throw → 500. `fetchOutletStatuses` is the reference.
- **Decorative annotation → best-effort.** Enrichment that's nice-to-have (e.g. Domain Rating from ahref-service). On failure, `console.warn` + serve the field as `null`. Never 500 the endpoint for it. `getDrStatus` in the route handlers is the reference (the ahref *client* stays fail-loud; the *route* catches and degrades).

**Before choosing fail-loud for a cross-service read, confirm the dependency is deployed in EVERY target environment.** A prod-only dependency + fail-loud read = the non-prod environment 500s the whole endpoint. `ahref-service` is **prod-only** (no staging); Railway private DNS is environment-scoped, so `ahref-service.railway.internal` does not resolve from the staging environment. That's exactly why the DR read is best-effort (degrades to `null` on staging, real values in prod).

## Domain Rating + traffic (ahref-service)

- DR **and** monthly organic traffic are owned by **ahref-service** (domain-keyed cache; it owns the Apify scrape spend). outlets-service stays cost-free — it reads ONLY ahref's CACHE endpoints, never the `*-compute` scrape endpoints.
- **discover** (`POST /orgs/outlets/discover`) fires `POST /orgs/domains/dr-compute` **non-blocking** (`.catch` + log) for the freshly-discovered domains (deduped across cycles) — a trigger, never awaited, never fails discover.
- **`GET /orgs/outlets` enriches each outlet ALWAYS-ON (no opt-in param)** with two nullable fields, keyed by the same normalized (www-stripped, lowercased) domain:
  - `domainRating` ← ahref `GET /orgs/domains/dr-status` (`latestValidDr`)
  - `trafficMonthlyAvg` ← ahref `GET /orgs/domains/traffic-history` (`trafficMonthlyAvg`)
- **The reads MUST be partial-tolerant at scale (12k+ domains).** `getDrStatusForEnrich` / `getTrafficForEnrich` (`services/ahref.ts`) chunk by URL length, run chunks at bounded concurrency (6), and tolerate failures **per chunk**: a chunk that fails its bounded transient-retry budget drops only ITS domains to `null` — never the whole set. This replaced an all-or-nothing `getDrStatusBestEffort(getDrStatus)` whose single-chunk throw nulled every outlet for a real 12k-domain brand. Do NOT reintroduce a wrap that returns `{}` on any error.
- **Chunk-level retry:** `fetchDomainCacheChunk` retries on THROWN transient transport errors (ahref cold-start / pool-saturation → `fetch failed` whose cause walks to ECONNRESET/ETIMEDOUT/ECONNREFUSED, or AbortSignal `TimeoutError`) — backoff 250/500/1000, idempotent GET so write-safe. An HTTP 5xx is a real answer (request reached ahref) and is NOT retried (fail-loud per chunk).
- **`GET /orgs/outlets/:id`** merges `domainRating` ALWAYS-ON (single domain) via `getDrStatus` (fail-loud client, route degrades to `null`). It does NOT add `trafficMonthlyAvg`.
- `null` (either field) = ahref has no cached/trustworthy value for that domain, OR that chunk stayed unreachable after retries.
- `discoverCycle` / `discoverOutletsInCategory` in `category-discovery.ts` return `{ inserted, domains }` so the route can collect discovered domains for the trigger. The `buffer/next` discovery path does NOT trigger dr-compute (follow-up, not yet wired).
- Env vars: `AHREF_SERVICE_URL`, `AHREF_SERVICE_API_KEY` (the key = ahref's own inbound key, shared).

## Pricing ingestion (bronze → silver → sell)

Per-article purchase pricing flows `outlet_price_sources` (bronze, raw notes, append-only) → `outlet_pricing` (silver, one row/outlet) → `sell_price_cents` (generated = `round(amount_cents × sales_multiplier)`, default ×2.0). `amount_cents` is the INTERNAL retail cost — never crosses the `/orgs` boundary. Two derivation paths feed the SAME silver table:

- **Messy notes → LLM extraction.** Journalist email replies / doc-sheet pastes go through `extractAndUpsertPricing` (`pricing.ts`), which calls the platform LLM (**Gemini Flash** — `provider:"google", model:"flash"`). Only path that should spend LLM tokens. Broker notes (`source_id` set) fan out one extraction per member outlet.
- **Structured catalogs → deterministic import, NO LLM.** A clean broker rate-card CSV is imported by `scripts/import-tremplin-catalog.ts` (`tsx scripts/... --file <csv> [--dry-run]`): pure column parse → `outlet_pricing` (`model='deterministic-import'`, `confidence=1.0`). Running the LLM per row over a clean price column would only burn ~Nk calls and risk hallucinating an exact price. Parse/map helpers live in `src/lib/tremplin-catalog.ts` (unit-tested); the script is thin DB glue (not part of the deployed service).

Idempotency: outlets upsert on domain; bronze guarded on `(outlet_id, captured_by)` (month-stamped marker, e.g. `tremplin-2026-06`) so re-imports append a fresh snapshot without dups; silver upserts on `outlet_id`. `sales_multiplier` is left out of every UPDATE so a manual per-outlet margin override survives re-derivation. The sensitive-topic 2–3× surcharge is a sell-side, per-order rule — NOT baked into `amount_cents`. Catalog DR/traffic/lang columns are ignored on import (DR stays ahref-service's).

## Editorial contacts (discovery + curated bronze + not_found)

The "who do we email for a paid-article rate card" pipeline ALREADY EXISTS — do NOT rebuild it. Two layers:

- **Org silver cache (scrape-derived).** `discoverEditorialEmails` (`services/editorial-emails.ts`) runs a 4-rung ladder (raw scrape contact/about → sitemap → JS-render → Google serper), scores + role-classifies, caches per `(org, domain)` 60d in `outlet_editorial_emails` + `outlet_editorial_email_lookups`, verifies deliverability via apify. Endpoints `POST /orgs/outlets/editorial-emails/discover[-batch]`. The rate-card outreach itself is `POST /orgs/outlets/price-requests` (`services/price-requests.ts`) → emails the editorial team, tracks the reply in `outlet_price_requests`.
  - **The ladder DEGRADES per rung — one rung's transient transport failure must NOT abort the whole discovery.** Each collection rung (Google, sitemap) runs through `collectTolerant`: a THROWN transient transport error (cold-start / pool-saturation / `/map` timeout on a huge site — `fetch failed`/`timed out`/ECONNRESET/ETIMEDOUT/ECONNREFUSED, classified by `lib/transient.ts::isTransientTransportError`) degrades THAT rung to empty and falls through to the next rung (so Path A timing out still lets Path B find the email). An **HTTP 5xx** from a provider is a real answer (request reached it) → NOT transient → propagates fail-loud (502), same rule as the ahref per-chunk retry. A non-transport error (LLM categorize, a bug) also propagates. Do NOT reintroduce a bare `await collectFromGoogle/Sitemap` that lets a single rung's timeout throw the whole ladder.
  - **`discovery_error` status = a rung stayed unreachable → "no email" is INCONCLUSIVE, not confirmed.** Distinct from `no_email_found`/`parked_dead` (real terminals, cached 60d): `discovery_error` is **NOT cached** (a 60-day cache of a timeout would suppress every retry) — the domain is left uncached so the next call re-attempts. `writeCache` is skipped for it in `discoverEditorialEmails`. The price-request flow surfaces it as a per-outlet `error` result (`No valid editorial email (discovery_error)`), never aborting the batch.
- **Global curated bronze (manually verified) — wins over the scrape cache.** `outlet_editorial_curation` (one verdict per outlet: `found` / `not_found`, permanent, org-agnostic) + `outlet_editorial_email_sources` (curated emails with provenance: `source_url`, `capture_method` page/search/social/manual, `confidence`, `captured_by`). Seeded via `POST /internal/editorial-emails/sources` (api-key only, no org) — `seedEditorialEmailSources` upserts the outlet by domain, records the verdict, and for `found` stores its emails. `discoverEditorialEmails` Rung 0 (`readCuratedEditorial`) checks this FIRST: a `found` verdict serves the curated emails, a `not_found` verdict serves terminal `no_email_found` WITHOUT scraping a known-dead domain (so we never re-call the ones with no reachable editorial email). Zero blast when empty (one indexed SELECT → 0 rows → falls through). Seed data + runner: `scripts/editorial-emails-seed.json` + `scripts/seed-editorial-emails.ts`.
- **Dashboard `not_found`.** `GET /orgs/outlets` (+ `/:id`) expose `editorialEmailStatus` (`found` | `not_found` | `null`) via a LEFT JOIN on `outlet_editorial_curation` — so the dashboard shows which outlets have no editorial contact. Decision: NO `sells_paid` column (the verdict is found/not_found only; whether an outlet sells paid articles is left to the reply, not pre-classified).

---
> Source: [shamanic-technologies/outlets-service](https://github.com/shamanic-technologies/outlets-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
