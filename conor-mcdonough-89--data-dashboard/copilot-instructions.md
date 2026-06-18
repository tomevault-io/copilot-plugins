## data-dashboard

> Password-gated, multi-sport seller analytics dashboard for SidelineSwap. For each sport (tennis, hockey, baseball, softball, golf, lacrosse) it shows the **top 100 sellers by 8-week GMV** with a weekly heat-map table, a US/Canada seller map, per-seller GMV trend chart, and five seller-insight tabs (Trajectory, AOV Leaders, Consistency, Specialists, Subscriptions). Data comes from either CSV upload or a direct **Metabase Sync** that runs the canonical SQL queries against the warehouse.

# SidelineSwap Seller Dashboard

Password-gated, multi-sport seller analytics dashboard for SidelineSwap. For each sport (tennis, hockey, baseball, softball, golf, lacrosse) it shows the **top 100 sellers by 8-week GMV** with a weekly heat-map table, a US/Canada seller map, per-seller GMV trend chart, and five seller-insight tabs (Trajectory, AOV Leaders, Consistency, Specialists, Subscriptions). Data comes from either CSV upload or a direct **Metabase Sync** that runs the canonical SQL queries against the warehouse.

## Tech stack

Vanilla HTML/CSS/JS, no build step, no framework. Libraries loaded from CDN: **Leaflet 1.9.4** (map), **Chart.js 4.4.0** (trends), **PapaParse 5.4.1** (CSV). **IndexedDB** persists per-sport data and a geocoder cache. Hosted on **Vercel** with one Edge Function for the Metabase proxy. A zero-dependency Node `server.js` is included as a self-host fallback.

## File map

| File | Role |
| --- | --- |
| `index.html` | Single-page shell. Contains the importer overlay, sync overlay, password gate, confirm dialog. Loads scripts in order: geocoder, insights, importer, metabase, dashboard. |
| `dashboard.js` | All UI rendering. `Dashboard.init()` is the entry point. Contains the `SyncUI` helper module that drives the Metabase sync overlay. |
| `importer.js` | CSV parsing + the shared post-parse pipeline `importSportFromArrays(name, sellers, swaps, onProgress)` used by both CSV and Metabase paths. Also handles JSON snapshot export/import. |
| `metabase.js` | Metabase config, six-category whitelist, two SQL templates, session-token auth (`POST /api/session`) with API-key fallback, `syncCategory` / `syncAll`. All HTTP goes through `/api/metabase/*`. |
| `insights.js` | Pure compute. `Insights.compute(sellers, swaps, numWeeks)` → per-seller insight objects keyed by seller_id. `Insights.getSpecialistCategories(swaps)` → top 10 `item_category_2` values for the Specialists tab. |
| `geocoder.js` | Owns the IndexedDB schema (object stores `sports` and `geo-cache`). Public API: `openDB`, `detectFormat`, `loadBundledData`, `resolve`, `resolveBatch`, `GEO_DATA_VERSION`. Resolves zips via memory cache → IndexedDB → bundled CSVs → Nominatim (rate-limited, 1 req/sec). |
| `data/us_zips.csv`, `data/ca_fsa.csv` | Bundled lat/lng tables (2024 Census Gazetteer for US, FSA centroids for CA). `GEO_DATA_VERSION = 2` — bumping it triggers a refresh of stored sport coordinates on next load. |
| `style.css` | All styles, hand-written, dark theme. Variables defined in `:root`. |
| `api/metabase-proxy.js` | Vercel **Edge function**. Streams every `/api/metabase/*` request to `${METABASE_URL}/*`. Auto-prepends `https://` if scheme missing, validates the URL upfront, includes the resolved target in upstream-error responses. |
| `vercel.json` | Single rewrite: `/api/metabase/:path*` → `/api/metabase-proxy?p=:path*`. |
| `server.js` | Zero-dependency Node fallback for self-hosting (serves static files + same proxy). Not used on Vercel. |
| `package.json` | No deps. `npm start` runs `server.js`. Requires Node ≥18. |

## Data model

### `sportData` (one row per sport in the `sports` IndexedDB store)

```
{
  id:              "hockey",                  // slug derived from name
  name:            "Hockey",                  // display name
  detectedSport:   "hockey",                  // item_category_1 value
  sellers:         [ <seller row>, ... ],     // ≤100 rows
  numWeeks:        8,
  insights:        { <seller_id>: <insight>, ... },
  specialistCats:  [ "ice-hockey-skates", ... ],
  zipCoords:       { "02115": { lat, lng }, "M5V": { lat, lng }, ... },
  mapConfig:       { center: [lat, lng], zoom: N },
  geoDataVersion:  2,
  importedAt:      "2024-…"
}
```

### Seller row (input from query 1 / GMV CSV)

`seller_id`, `gmv_rank`, `total_gmv`, `week_1`…`week_N`, `ship_from_state`, `ship_from_zip`, `trade_in_account` (bool), `power_seller` (bool), `pro_seller` (bool), `pro_plus_seller` (bool), `category` (= `item_category_1`).

### Swap row (input from query 2 / Swaps CSV)

`paid_at`, `buyer_join_date`, `buyer_city`, `buyer_state`, `buyer_purchase_date`, `seller_id`, `cash_offer_amount`, `item_category_2`, `item_condition`. The repeat-buyer heuristic in `insights.js` compares `buyer_purchase_date` to `paid_at` — if the buyer's first-purchase date predates this swap's `paid_at`, it's a returning buyer.

## Key flows

**CSV import.** User clicks `+` tab → drops two CSVs → `parseCSV` (PapaParse) → `importSportFromArrays(name, sellers, swaps)` → `Insights.compute` → `Geocoder.loadBundledData` + `Geocoder.resolveBatch` → `saveSport` to IndexedDB → `Dashboard.onSportImported` re-renders.

**Metabase sync.** User clicks **Sync** → `SyncUI.open()` shows the overlay → `Metabase.syncAll(categoryIds, onProgress)` iterates each category sequentially. Per category: `Metabase.login()` issues `POST /api/session` (cached in `localStorage`), then `runQueryRowsCols` runs the Top-100 query via `/api/dataset` (≤100 rows) and `runQueryUnbounded` runs the 6-month swaps query via `/api/dataset/json` (no row cap). Both responses go through `importSportFromArrays`. After the loop, `Dashboard.onSportsSynced(results)` updates state and switches to the first synced sport. 401/403 from upstream triggers a single re-login + retry.

**Per-sport refresh.** Each sport tab gets a ↻ icon (only shown when `sport.detectedSport` is one of the six known categories). Clicking it calls `SyncUI.open([sport.detectedSport])`, preselecting just that one category.

## Storage

- **IndexedDB** (`sls-dashboard`): store `sports` (keyed by `id`) holds the `sportData` objects. Store `geo-cache` (keyed by zip/FSA `code`) holds resolved coordinates from the Nominatim fallback path.
- **localStorage**: `sls-active-sport` (last-viewed sport id), `sls-metabase-config` (`{ authMode, username, password, apiKey, databaseId }`), `sls-metabase-session` (`{ token, ts }`).
- **sessionStorage**: `sls-auth` = `'1'` once the password gate is unlocked for the session.

## Deployment

### Vercel (production)

1. Push to the deployment branch.
2. Vercel → Project → Settings → Environment Variables → set **`METABASE_URL`** to the Metabase host (e.g. `https://metabase.example.com`). Apply to Production and Preview.
3. Redeploy so the env var is live.
4. The Edge function at `api/metabase-proxy.js` handles `/api/metabase/*` via the rewrite in `vercel.json`.

### Self-host

```
METABASE_URL=https://metabase.example.com PORT=8080 npm start
```

`server.js` serves the static files and proxies `/api/metabase/*` to `METABASE_URL`. Same browser config works against either host.

## Categories

Defined as `Metabase.CATEGORIES` in `metabase.js`:

| `item_category_1` | Display name |
| --- | --- |
| `tennis-racquet-sports` | Tennis |
| `hockey` | Hockey |
| `baseball` | Baseball |
| `softball` | Softball |
| `golf` | Golf |
| `lacrosse` | Lacrosse |

The category id is inlined into the SQL templates after a whitelist check, so it's safe from injection.

## Constraints / gotchas

- **SQL is BigQuery dialect** (`DATE_DIFF`, `DATE_SUB`, backticked table refs `` `rails.exports_swaps` ``). Switching warehouses requires rewriting the templates in `metabase.js`.
- **The dashboard password is hardcoded** in `index.html` (currently `shippinglogisticsguy`). Change it there.
- **Metabase username + password are stored in `localStorage`.** SSO-only Metabase instances must use the API-key path via the auth-mode toggle.
- **CORS is sidestepped by the proxy** — Metabase admin does not need to touch `MB_CORS_ALLOWED_HOSTS`. Direct browser-to-Metabase fetches are not used.
- **Swap responses can be tens of MB.** Handled via `/api/dataset/json` (no 2,000-row cap) and Edge-runtime streaming on Vercel.
- **`switchSport` short-circuits when the id matches the active sport.** When refreshing the active sport in place, `Dashboard.onSportImported` force-rebuilds via a state reset; don't add a callsite that calls `switchSport(activeSportId)` directly.

## Common tasks

**Add a new sport category.** Add an entry to `Metabase.CATEGORIES` in `metabase.js` (use the `item_category_1` slug from the warehouse and a display name). The sport tab, sync UI, and SQL templates pick it up automatically.

**Change the dashboard password.** Edit the `PASSWORD` constant in the inline script at the bottom of `index.html`.

**Point at a different Metabase host.** Change `METABASE_URL` in Vercel project settings (or in the env when running `server.js`) and redeploy. The browser-side config has no URL field — the server holds the upstream URL.

**Debug a failed sync.** From devtools console on a deployed site:
```
fetch('/api/metabase/api/health').then(r => r.text()).then(console.log)
```
- 404 `NOT_FOUND iad1::…` → proxy file isn't on this deployment (routing/branch issue).
- `{"error":"METABASE_URL env var is not set..."}` → set the env var and redeploy.
- `{"error":"Upstream Metabase error: …"}` → the proxy is wired but can't reach Metabase (target URL is included in the message).
- `{"status":"ok"}` (or similar Metabase response) → end-to-end works; any further failures are auth/SQL related.

**Run locally without Vercel.**
```
METABASE_URL=https://metabase.example.com node server.js
# open http://127.0.0.1:8080
```

**Inspect / clear stored data.** Browser devtools → Application → IndexedDB → `sls-dashboard`. Object stores `sports` and `geo-cache`. localStorage holds the auth/session config. Wipe all to fully reset.

---
> Source: [conor-mcdonough-89/data-dashboard](https://github.com/conor-mcdonough-89/data-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
