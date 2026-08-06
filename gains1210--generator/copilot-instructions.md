## generator

> Internal web tool for a Viennese fitness supplement store (MuscleZone).

# MuscleZone Pricetag Generator — AI Handoff Context

## What This Is

Internal web tool for a Viennese fitness supplement store (MuscleZone).
Purpose: generate A4 price-tag sheets (print/PDF) sourced from the helloCash POS system.
Live at **https://pricetag.musclezone.at** (Vercel, auto-deploys from `main`).

Replaces a PHP/HTML single-page app. Same UX, modern stack.

---

## Stack

| Layer | Tech |
|---|---|
| Framework | Next.js 16 App Router + React 19 + TypeScript |
| Styling | Tailwind CSS v4 |
| Database | Supabase Postgres |
| Auth | Supabase Auth — **currently DISABLED** (see proxy.ts) |
| Deployment | Vercel (branch `main` → auto-deploy) |
| POS source | helloCash REST API v1 (read-only, server-side only) |

---

## Breaking: This Is NOT Standard Next.js

<!-- BEGIN:nextjs-agent-rules -->
Next.js 16 has breaking changes from training data:
- **Middleware file is `src/proxy.ts`**, NOT `middleware.ts`
- Read `node_modules/next/dist/docs/` before changing routing/middleware
- App Router is used throughout (`src/app/`)
<!-- END:nextjs-agent-rules -->

---

## Auth Status: DISABLED

`src/proxy.ts` passes all requests through without auth checks:
```typescript
export function proxy() { return NextResponse.next(); }
export const config = { matcher: ['/((?!_next/static|_next/image|favicon.ico).*)'] };
```
This is intentional — the store is internal and the client prefers no login.
To re-enable: restore Supabase session check in proxy.ts.

---

## Environment Variables (set in Vercel, NOT in .env.local)

| Variable | Visibility | Purpose |
|---|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | Client + Server | Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Client + Server | Anon/publishable key (RLS) |
| `SUPABASE_SECRET_KEY` | **Server only** | Service-role key (admin client, sync) |
| `HELLOCASH_API_TOKEN` | **Server only** | helloCash Bearer token (read-only) |
| `HELLOCASH_API_BASE_URL` | Server | Default: `https://api.hellocash.business/api/v1` |

**Important:** Code accepts both `NEXT_PUBLIC_SUPABASE_ANON_KEY` and
`NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` (fallback) because the var was renamed.
`NEXT_PUBLIC_*` vars are embedded at build time — a rebuild is needed after changing them.

---

## Database Schema

Migrations live in `supabase/migrations/` (additive, non-destructive).
Migrations 0001, 0002, 0003 are **already applied** in the live Supabase project.

| Table / View | Role |
|---|---|
| `products` | Mirror of helloCash articles (managed by sync) |
| `product_overrides` | Local edits — NEVER overwritten by sync |
| `categories` | Category lookup (name → id) |
| `sync_cache` | SHA-256 hash per external article for incremental sync |
| `sync_runs` | Sync log: created / updated / unchanged / skipped counts |
| `label_settings` | Label layout, fonts, dimensions |
| `label_selection_sets` / `label_selection_items` | Saved product selections |
| `effective_products` (view) | `coalesce(override, source)` — what the UI reads |

`effective_products` view merge logic: for each field (name, price, category, EAN, icon),
the override wins over the source value.

### RLS Policies (migration 0003)

Since auth is disabled, anon users must be able to read/write:
- `product_overrides`: anon full access
- `sync_cache`: anon full access
- `label_selection_sets` / `label_selection_items`: anon full access
- `sync_runs`: anon read-only; service_role write-only

**Admin client** (`getSupabaseAdmin()` using `SUPABASE_SECRET_KEY`) bypasses RLS.
Use it for: sync writes, reading sync_runs, saving overrides, label selections, settings.
Do NOT use `createClient()` (anon) for mutations — it goes through RLS.

---

## Supabase Client Usage Rules

```typescript
// Server-side admin (bypasses RLS) — for sync, mutations, reads that need all rows
import { getSupabaseAdmin } from '@/lib/supabase/admin';
const supabase = getSupabaseAdmin();

// Server-side anon (subject to RLS) — for public reads only
import { createClient } from '@/lib/supabase/server';
const supabase = createClient();
```

`getSupabaseAdmin()` is **lazy** — throws at call time, not at build time, when `SUPABASE_SECRET_KEY` is missing.
`createClient()` must be called **inside try/catch** in server components (Next.js 16 lint rule).
JSX must **not** be rendered inside try/catch blocks (Next.js 16 lint rule).

### Supabase Row Limit

Supabase returns max 1000 rows per query by default. Use `.range()` pagination in a loop:
```typescript
let from = 0;
const CHUNK = 1000;
while (true) {
  const { data } = await supabase.from('effective_products').select('*').range(from, from + CHUNK - 1);
  all.push(...data ?? []);
  if ((data?.length ?? 0) < CHUNK) break;
  from += CHUNK;
}
```
See `src/lib/products/queries.ts` for the full implementation.

---

## helloCash API

Base URL: `https://api.hellocash.business/api/v1`
Auth: `Authorization: Bearer <HELLOCASH_API_TOKEN>`
Mode: **read-only** — never write back to helloCash.

### Pagination (confirmed from live API debug)

First call: `GET /articles` (no params)
Response shape:
```json
{
  "articles": [...],
  "count": "2031",   // STRING, not number — total article count
  "limit": 250,      // page size
  "offset": 1        // 1-based page number (NOT item offset)
}
```

Pages 2+: `GET /articles?limit=250&offset=2`, `?limit=250&offset=3`, etc.
`offset` is a **1-based page number**, not an item byte/row offset.
Stop when: `items.length === 0`, or all items are duplicates, or page > 200.

### Article Field Names (confirmed from live API)

| helloCash field | Our DB field | Notes |
|---|---|---|
| `article_id` | `external_id` | Primary identifier |
| `article_name` | `name` | Full name e.g. "DY Nutrition - Shadowhey Vanilla Flavour" |
| `article_gross_sellingPrice` | `price_cents` | Decimal string e.g. `"14.9"` → 1490 cents |
| `article_eanCode` | `ean` | EAN/barcode |
| `article_comment` | `quantity` | Content/size e.g. "100 Kapseln", "908g" |
| `article_unit` | `unit` | Unit of sale e.g. "Stück" |
| `article_category_id` | (numeric, ignored) | Numeric category ID only — no name available |

**Manufacturer**: helloCash has no manufacturer field. Extract from `article_name` prefix:
`"DY Nutrition - Shadowhey Vanilla Flavour"` → manufacturer = `"DY Nutrition"`
Split on first `" - "` (space-dash-space).

**Category names**: All helloCash category/group endpoints return 400.
Only `article_category_id` (numeric) is available. Set `category_name = null` in sync.
Categories from the legacy data import remain intact in the `categories` table.

### helloCash Client: `src/lib/hellocash/client.ts`

Implements `fetchArticles()` with full pagination. Correctly handles:
- Parsing `count` as string: `parseInt(String(rawObj['count'] ?? '0'), 10)`
- 1-based page offset: `?limit=250&offset=2`, `?limit=250&offset=3`, …
- Dedup by `article_id` to prevent duplicates across pages

### Article Mapping: `src/lib/hellocash/mapArticle.ts`

Maps a raw helloCash article to `NormalizedArticle`. Uses the confirmed field names above.
`manufacturer` is extracted from the name prefix before `" - "`.

---

## Sync Flow

`POST /api/hellocash/sync` → `src/lib/sync/syncHellocash.ts`

1. Fetch all articles from helloCash (paginated, typically ~9 pages × 250 articles = ~2031)
2. For each article: compute SHA-256 hash of payload
3. Compare to `sync_cache` hash — skip if unchanged
4. Upsert changed/new articles into `products` (200-row batches)
5. Mark missing articles as `active = false` (never deleted)
6. Update `last_seen_at` timestamps (500-row batches)
7. Insert row into `sync_runs` with counts

Route has `maxDuration = 60` (Vercel serverless limit).
All DB writes use `getSupabaseAdmin()`.

Sync is triggered from the `/sync` page (button click) or via direct POST.

### Sync Diagnostics (migration 0004)

`fetchArticles()` returns `{ articles, totalAvailable, pagesFetched }`. `syncHellocash`
records these plus `missing_count` and `duration_ms` into `sync_runs`. The `/sync` page shows:
- **Bestand**: live product inventory (total / active / inactive / hellocash / manual / overrides)
- **Abruf**: helloCash total vs. fetched, pages, duration — with a reconciliation banner that
  flags when `fetched_count < total_available` (the root cause of "only 1298 products")
- **Verarbeitet**: created / updated / unchanged (cache) / skipped / missing
- **Verlauf**: last 20 runs with all of the above; "Verlauf leeren" clears `sync_runs` only

Rate limit: helloCash allows ~60 req/min. A full catalogue (~2031 articles @ 250/page) is
only ~9 requests, so rate limiting is not a bottleneck for a normal sync.

---

## App Routes

| Route | Description |
|---|---|
| `/` | **Main page (index)** — full generator: product picker + live preview + inline edit |
| `/products` | Product list; click a row to edit inline; "Neues Produkt" to create manual products |
| `/labels` | Same generator as `/`, kept as its own nav item |
| `/settings` | Label layout settings with live preview (format fixed A4) |
| `/sync` | Sync tool: inventory, diagnostics, history |

Root `/` is the generator (not a redirect). `src/app/page.tsx` renders `LabelsWorkbench` directly.
There is **no** `/products/[id]` detail page anymore — all editing is inline via a modal.

---

## UI / UX Conventions (rebuilt June 2026)

- **Light-first**: `globals.css` defines `@custom-variant dark (&:where(.dark, .dark *))` so
  Tailwind `dark:` utilities only apply under a `.dark` class (never added). This stops the app
  from turning dark on phones whose OS is in dark mode. Accent colour is **blue-600**.
- **Inline editing**: `ProductEditDialog` (modal) edits any product. It always writes to
  `product_overrides` (override-wins), so edits survive the next sync. Used by both the products
  table and the generator picker (pencil icon / "Dieses Produkt bearbeiten").
- **Manual products**: `createManualProduct` inserts a row into `products` with `source='manual'`
  and a random `source_id`. They appear via `effective_products` like any product, are editable,
  and `deleteManualProduct` removes them (refuses to delete `source='hellocash'` rows).
- **Marke (brand)** = `manufacturer`, shown as its own editable column in both tables.
- **Selection persistence**: `SelectionContext` (mounted in root `app/layout.tsx`) holds the
  selected product ids; it survives navigation and is mirrored to `localStorage`. The old
  "Auswahl speichern für Aktionszeitraum" feature and `label_selection_*` UI were removed.

## Performance / Caching

Navigating used to refetch all ~2000 products server-side on every click. Now:
- `getCachedProducts` / `getCachedCategories` (`queries.ts`) and `getCachedSettings`
  (`settings.ts`) wrap the heavy reads in `unstable_cache` tagged `PRODUCTS_TAG` / `settings`
  (`revalidate: 300`).
- **Next 16 cache invalidation**: server actions use `updateTag(tag)` (read-your-own-writes,
  single arg, Server-Action-only). Route handlers (the sync route) use
  `revalidateTag(tag, 'max')` — the **two-arg** form; single-arg `revalidateTag` is deprecated
  and fails the TS build.

---

## Key Components

- `src/components/labels/LabelsWorkbench.tsx` — generator; selection via `useSelection()`,
  hosts `ProductEditDialog`, live `PriceTag` preview + zoom.
- `src/components/labels/ProductPicker.tsx` — picker table (Produktname, Marke, Menge, Einheit,
  Preis + edit pencil); row click toggles select + sets preview.
- `src/components/labels/PriceTagPreview.tsx` — exact label design (black header = manufacturer,
  bold name, quantity, big price w/ superscript cents + €, footer "Price per: unit"). 5.03×3.75 cm.
- `src/components/products/ProductsTable.tsx` / `ProductEditDialog.tsx` — list + inline edit/create.
- `src/components/settings/LabelSettingsForm.tsx` — categorized settings + live preview, fixed A4.
- `src/components/sync/SyncPanel.tsx` / `ClearHistoryButton.tsx` — sync action + diagnostics.
- `src/components/layout/AppNav.tsx` — brand → "/", hamburger menu on mobile, blue active state.

---

## Debug Endpoint (TEMPORARY — remove after verifying sync)

`GET /api/hellocash/debug` — returns raw API response shape, pagination fields, confirmed
field names, category endpoint probe results, and page 2 item count.
File: `src/app/api/hellocash/debug/route.ts`

---

## Known Issues / Pending Tasks

1. **Verify sync after mapArticle.ts fix**: Field names in mapArticle.ts were previously wrong
   (camelCase guesses vs confirmed `article_*` names). Now fixed. Run a full sync and check
   products have correct names, prices, and quantities.

2. **Category names are null**: helloCash provides no category name endpoint.
   Products synced from helloCash will have `category_name = null`.
   Existing categories from the legacy import are unaffected.

3. **Remove debug endpoint** after confirming sync works end-to-end.

4. **~2031 products expected after sync**: helloCash has 2031 articles as of June 2026.
   Pre-import populated ~1298. A fresh sync should upsert the remainder.

---

## Security Rules

- **Never commit**: `.env`, `.env.local`, `SUPABASE_SECRET_KEY`, `HELLOCASH_API_TOKEN`
- Secret/service-role key and helloCash token are **server-side only**
- Client code sees only `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- helloCash is **read-only** — never write back to it

---

## Migrations

Applied in production: `0001`, `0002`, `0003`.

**`0004_sync_run_diagnostics.sql` — PENDING.** Must be run in the Supabase SQL Editor **before**
deploying the new sync code (it adds `fetched_count`, `total_available`, `pages_fetched`,
`missing_count`, `duration_ms` to `sync_runs`; the run-logging insert fails without it). Additive only.

Inline editing and manual products need **no** new migration — `product_overrides` already exists
and manual products are plain `products` rows (`source='manual'`) written via the admin client.

New migrations go in `supabase/migrations/` and must be applied manually in Supabase SQL Editor
(no local Supabase CLI is configured).

---
> Source: [GAINS1210/generator](https://github.com/GAINS1210/generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
