## property-dashboard

> Automated property search and CRM dashboard for finding a home near Yosemite (Mariposa County / Madera County, CA).

# Yosemite Property Tracker

Automated property search and CRM dashboard for finding a home near Yosemite (Mariposa County / Madera County, CA).

## Architecture

Two Cloudflare projects, both bound to the same D1 database:

- **`collector/`** -- Cloudflare Worker. Hourly cron fetches listings from RealtyAPI (Zillow data), upserts into D1. Also serves the internal REST API that the dashboard's Pages Functions proxy to.
- **`dashboard/`** -- Cloudflare Pages (React + Vite). Table view for triaging properties through stages. Pages Functions in `dashboard/functions/` query D1 directly for reads and proxy writes to the collector via service binding.

**IMPORTANT: Dual query locations.** GET endpoints for listings and stats have their own SQL queries in BOTH `collector/src/routes.js` AND `dashboard/functions/api/`. The Pages Functions do NOT proxy GET requests to the collector — they query D1 directly. Any query changes (new fields, joins, filters) MUST be applied to both locations or the dashboard will not reflect the change.

**Error handling pattern:** All Pages Functions in `dashboard/functions/api/` wrap their handler body in try/catch and return `Response.json({ error: err.message }, { status: 500 })` on failure. Service binding calls (PATCH/DELETE/POST to collector) have 8-second timeouts. The frontend (`dashboard/src/api.js`) enforces a 10-second fetch timeout via AbortController with one automatic retry for timeouts, network errors, and 5xx responses. Overlapping data refreshes are cancelled via AbortController (only the latest wins). D1 has intermittent latency spikes — maintain these safeguards when adding new endpoints.

**Cloudflare Access session handling:** The dashboard production URL is behind Cloudflare Access (WARP-based auth, 2-week session). The frontend API layer uses `redirect: "manual"` on fetch and detects opaque redirects / HTML responses as signs of an expired Access session, then triggers `window.location.reload()` to re-authenticate through the browser. This prevents JS fetch calls from silently hanging when the session expires.

## Cloudflare Resources

| Resource | Name | ID |
|----------|------|----|
| Worker | yosemite-property-collector | -- |
| Pages | yosemite-property-dashboard | -- |
| D1 Database | yosemite-properties | *(set after `wrangler d1 create`)* |

The dashboard binds to the collector via a service binding (`COLLECTOR`). Both projects bind to D1 as `DB`.

## RealtyAPI (Zillow Data Source)

- Endpoint: `GET https://zillow.realtyapi.io/search/byurl?url={zillow_url}`
- Auth: `x-realtyapi-key` header (NOT Bearer)
- Plan: $20/mo, 20,000 requests/month, no rate limits
- Zillow URL format: `https://www.zillow.com/homes/{zip}_rb/0-{maxPrice}_price/`
- Pagination: append `{page}_p/` to the URL (e.g. `.../0-1100000_price/2_p/`)
- Results per page: 41
- The API is a third-party Zillow scraper. It returns `"Error: list index out of range"` in the `message` field (not `error`) during intermittent outages. The collector has retry logic (3 attempts, 1s backoff) to handle this.

## Secrets

The RealtyAPI key is stored as a Cloudflare Worker secret (`REALTY_API_KEY`). Set it via:

```sh
# Pipe your API key into wrangler so it never appears in shell history
echo "YOUR_REALTY_API_KEY" | cd collector && npx wrangler secret put REALTY_API_KEY
# Or from a secrets manager (e.g. 1Password CLI):
# op read "op://your-vault/realty-api-key/api_key" | npx wrangler secret put REALTY_API_KEY
```

## Cloudflare Access Service Token

The collector Worker is behind Cloudflare Access. To make authenticated requests (e.g. trigger `/test-run`), use a Cloudflare Access service token. Pass the `CF-Access-Client-Id` and `CF-Access-Client-Secret` headers:

```sh
curl -s \
  -H "CF-Access-Client-Id: $CF_CLIENT_ID" \
  -H "CF-Access-Client-Secret: $CF_CLIENT_SECRET" \
  https://<your-worker-subdomain>.workers.dev/test-run
```

## Deploying

**IMPORTANT: The dashboard is a Vite/React app. You MUST run `npm run build` before `wrangler pages deploy`, otherwise you deploy stale assets from the previous `dist/`.** Also pass `--branch main` to target production — without it, wrangler still deploys to production but the deploy URL looks like a preview and can cause confusion.

```sh
# Collector (from collector/)
npx wrangler deploy

# Dashboard (from dashboard/)
npm run build && npx wrangler pages deploy --branch main
```

The collector cron is defined in `collector/wrangler.toml`. Currently set to every 6 hours (`0 */6 * * *`).

## D1 Schema

Four tables: `listings`, `price_history`, `status_history`, `collection_runs`. Schema is in `collector/schema.sql`.

To run migrations against remote D1:
```sh
cd collector && npx wrangler d1 execute yosemite-properties --remote --file=schema.sql
```

To run ad-hoc queries:
```sh
cd collector && npx wrangler d1 execute yosemite-properties --remote --command "SELECT ..."
```

D1 commands must run from the `collector/` directory (needs wrangler.toml context).

## Search Configuration

`collector/src/config.js` defines the zip codes, max price ($1,100,000), and property types. Current zips: 95345 (Midpines), 95338 (Mariposa/Bootjack/Mount Bullion), 95318 (El Portal), 93644 (Oakhurst), 93601 (Ahwahnee), 93623 (Fish Camp), 95389 (Yosemite West), 95305 (Big Oak Flat), 95321 (Groveland).

## Dashboard Stages

Defined in `dashboard/src/config.js`:
1. **Inbox** -- no action taken yet
2. **Consider** -- worth a closer look
3. **Untoured** -- want to schedule a tour
4. **Toured** -- toured in person
5. **Pass** -- not interested

## Collector Behavior

- Updates never overwrite user fields (stage, notes, tour_date, tour_notes)
- Tracks price changes and status changes in history tables
- Off-market detection: listings not seen in 48 hours are marked "gone"
- Listings that reappear after going off-market are "resurrected"
- `/test-run` endpoint triggers a manual collection (bypasses 30-min rate limit). Requires Cloudflare Access auth (see service token section above)
- Config changes (zip codes, price, etc.) require redeploying the collector to take effect

## Dashboard Features

- Table view (pipeline/kanban view removed)
- Bulk select + bulk stage assignment
- Off-market listings hidden by default, toggle to show (dimmed)
- Detail panel with Zillow link, Google Maps, driving directions to Mariposa, El Portal Gate, Big Oak Flat entrance, and South Entrance
- Price history and status history display
- DOM (Days on Market) color coding: fresh (0-21d), leverage (21-60d), negotiate (60-90d), why? (91+)
- Search, sort, stage filter
- Price drop filter (clickable stat in top bar) and purple price highlighting
- Detail panel auto-closes after stage change for faster triage
- Auto-refreshes data when returning to an idle tab (60s threshold)

## Local Dev

`localhost` dev server has no data — it connects to a local D1, not the remote database. Don't bother testing UI changes locally. Build, deploy to Pages, and test on the live site.

---
> Source: [MountainsCalling-me/property-dashboard](https://github.com/MountainsCalling-me/property-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
