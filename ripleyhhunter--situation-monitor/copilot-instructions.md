## situation-monitor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Situation Monitor is a real-time situation monitoring dashboard. The codebase
is **multi-region**: the backend schedules and fetches **all** regions in
`regions/index.ts` `allRegions` concurrently, broadcasts every region's data
to every SSE client, and the frontend picks which region to surface (header
switcher, persisted in localStorage). The `REGION` / `PUBLIC_REGION` env vars
only set the *default/initial* selection. Current regions:

- **`dc`** — Washington, DC + DMV (PulsePoint EMS1205, MD CHART, MDOT WZDx,
  VDOT NoVA feeds, DC Open Data, DC 311, WMATA, AlertDC, ShotSpotter,
  MoCo/PG crime, OpenMHz DC Fire & EMS scanner audio)
- **`boise`** — Boise / Ada County / Treasure Valley (PulsePoint EMS1169
  ACCESS, BPD Crimes ArcGIS, Ada County CrimeMapper, ITD WZDx + ACHD RITA
  roadwork, Idaho 511 live incidents + Waze reports, Idaho 511 cameras,
  NWS Boise webcams)

Shared (region-agnostic) fetchers: NWS weather alerts, Open-Meteo current
conditions, AirNow, OpenSky aircraft, news RSS, NIFC WFIGS wildfires, USGS
earthquakes, NWS river gauges. Each is instantiated
per-region by the pack file with the region's config (zones, timezone, bbox,
feeds) passed to its constructor — no fetcher reads a global region.

## Region Pack Architecture

Each region exports a `RegionPack` from `regions/<id>.ts` (see
`regions/types.ts` for the contract). A pack bundles:
- **Static config**: `city`/`state`/`timezone`, `defaultCenter`, `openSkyBounds`, `nwsZones`, `sourcesWithCompleteListing`
- **Fetcher arrays** grouped by scheduling profile: `cameraFetchers`, `trafficIncidentFetchers`, `crimeFetchers`, `shotspotterFetchers`, `emergencyAlertFetchers`
- **Singleton-or-null fetchers**: `pulsePointFetcher`, `transitFetcher`, `scannerFetcher`, `twitterFetcher`
- **News config**: RSS feeds + region-specific area keywords / location regex patterns

`regions/index.ts` exports `allRegions` (what the aggregator iterates —
it has zero hardcoded region knowledge), `regionsById`, and
`defaultRegionId` (from `process.env.REGION`, a frontend-default hint only).

Incident lifecycle notes:
- Sources listed in `sourcesWithCompleteListing` are snapshot feeds: absence
  from a successful poll implies cleared (including empty snapshots — their
  fetchers declare `incidentSource` on `BaseFetcher` so this works when a
  feed empties). They are exempt from the age-based cleanup sweep.
- Other sources age out per `expirationMs` in `aggregator.cleanupStaleIncidents`;
  a fetcher's own fetch-window filter must stay inside that window or records
  will clear/re-add in a loop (see bpd-crime / dc-shotspotter for the pattern).
- Normalizers should derive `updatedAt` from feed fields, never `now` — the
  aggregator diffs on `updatedAt` to decide what to re-broadcast.
- Geocoding is region-scoped: `geocache.geocode(address, {city, state, center})`.

## Key Limitations & Solutions

| Limitation | Solution |
|------------|----------|
| DC Metro Police radios encrypted since 2011 | No police scanner data available |
| Boise PD on SWIRC P25 Phase II (some channels encrypted) | Same — link out to RadioReference / PulsePoint instead |
| Neither city publishes raw Fire/EMS CAD | **PulsePoint scraping** via Playwright (DC: EMS1205, Boise: EMS1169 / ACCESS) |
| Twitter API costs $100/month | Optional `TWITTER_BEARER_TOKEN` (DC `@dcfireems` only) |
| Boise crime data lags ~1 month | bpd-crime uses a 60-day fetch window + matching 60-day expiry; near-real-time `BPD_CallsForService` exists but not yet wired |
| DC ShotSpotter feed appears stale (no new detections since ~2026-04) | 30-day fetch window — the layer is honestly empty until the feed revives |
| ITD camera API requires a key | Skipped — Boise's `cameraFetchers` only ships curated NWS Boise airport cams |
| OpenMHz blocks Node's TLS fingerprint (403) while serving curl/browsers | Scanner fetcher shells out to the system curl; DC Fire & EMS call audio plays in the scanner panel. No Idaho systems exist on OpenMHz, so Boise keeps link-outs |

## Build & Development Commands

This is a Turborepo + npm workspaces monorepo. Workspace package names: `@situation-monitor/frontend`, `@situation-monitor/backend`.

**Windows shortcut:** `start-monitor.bat` at the repo root (personal,
gitignored — exists only on the owner's machine) does `docker-compose up -d`
+ `npm run dev`, and runs `docker-compose down` on exit.

**Note on `npm run clean`:** the per-workspace `clean` scripts use POSIX `rm -rf` and will fail in plain PowerShell — use Git Bash / WSL or delete `dist/`, `.svelte-kit/`, `build/`, `node_modules/` manually.

```bash
# From repo root
npm install              # Install all dependencies
npx playwright install chromium   # One-time: browser for PulsePoint scraping
npm run dev              # Start frontend + backend with hot-reload (via turbo)
npm run docker:up        # Start Redis (OPTIONAL — falls back to in-memory cache)
npm run docker:down      # Stop Redis
npm run build            # Production build (both workspaces)
npm run test             # Run tests once and exit (both workspaces)
npm run lint             # Lint both workspaces

# Scope a command to one workspace from the repo root
npm run dev   --workspace=@situation-monitor/backend
npm run build --workspace=@situation-monitor/frontend

# Backend only (from packages/backend)
npm run dev              # tsx watch src/index.ts
npm run build            # Compile TypeScript -> dist/
npm run start            # Run production build (node dist/index.js)
npm run test             # vitest run (single pass)
npm run test:watch       # vitest watch mode

# Run a single backend test
cd packages/backend
npx vitest run src/config.test.ts                     # one file
npx vitest run -t "return cached data"                # by test name (substring match)

# The PulsePoint E2E (live Chromium against web.pulsepoint.org) is skipped
# unless RUN_E2E is set:
RUN_E2E=1 npx vitest run src/fetchers/pulsepoint.test.ts

# Frontend only (from packages/frontend)
npm run dev              # Vite dev server (port 5173, proxies /api -> :3000)
npm run build            # Static build to build/
npm run preview          # Preview production build
npm run check            # svelte-check (types + Svelte template checks)
npm run lint             # eslint
npm run test             # vitest run (stores/utils unit tests)
```

## Architecture

### Monorepo Structure
- **packages/frontend**: SvelteKit 2.x + Svelte 5, Leaflet.js, TailwindCSS
- **packages/backend**: Node.js 20, Express, Redis (optional), Playwright

### Backend Data Flow
```
External APIs → Fetchers (node-cron scheduled, per region) → Normalizers
  → in-memory region state (+ Redis snapshot of active incidents) → SSE Broadcast → Frontend
```
`database.ts` is an in-memory mirror; durable storage is
`services/history.ts` (SQLite via better-sqlite3, `data/history.db`,
180-day retention): every incident is journaled there, `/api/history/*`
serves trend aggregates, and on startup active incidents restore from the
Redis `incidents:active:<region>` snapshot or — when Redis is absent —
from SQLite.

### Key Files

**Backend:**
| File | Purpose |
|------|---------|
| `src/services/aggregator.ts` | Orchestrates all fetchers per region, manages state, cleanup sweeps, SSE broadcasting |
| `src/services/scheduler.ts` | Cron-based job scheduling |
| `src/services/sse.ts` | Server-Sent Events broadcasting + per-client aircraft-region preferences |
| `src/services/database.ts` | In-memory Map mirror (fast lookups only) |
| `src/services/history.ts` | Durable SQLite incident history (better-sqlite3): trends aggregates + Redis-less restart restore |
| `src/services/geocache.ts` | Region-scoped Nominatim geocoding with Redis-persisted cache |
| `src/regions/dc.ts`, `src/regions/boise.ts` | **The authoritative fetcher registry** — aggregator imports no fetchers itself |
| `src/fetchers/*.ts` | Individual API integrations (~30 modules — the region packs are the authoritative wiring list) |
| `src/routes/*.ts` | Express route handlers (health, incidents, cameras, weather, aqi, news, events) |
| `src/middleware/cors.ts` | Production CORS allowlist; dev is wide-open |

**Frontend:**
| File | Purpose |
|------|---------|
| `src/lib/stores/*.ts` | Svelte stores (incidents, cameras, weather, filters, location) |
| `src/lib/services/sse.ts` | SSE client with auto-reconnect |
| `src/lib/components/map/MapContainer.svelte` | Leaflet map with clustering & heatmap |
| `src/lib/components/ui/Header.svelte` | Header with weather, metro delays, AQI |
| `src/lib/components/ui/SearchBar.svelte` | Address search with Nominatim geocoding |

### All Data Fetchers

Authoritative list: the fetcher wiring in `packages/backend/src/regions/dc.ts`
and `regions/boise.ts` (the aggregator imports no fetchers itself). 22 fetcher
modules exist; the table below covers the notable ones.

| Fetcher | Source | Type | Interval | Notes |
|---------|--------|------|----------|-------|
| `pulsepoint.ts` | PulsePoint | Fire/EMS incidents | 2 min | Playwright headless browser; only runs when SSE clients connected |
| `mdchart-cameras.ts` | MD CHART | Traffic cameras | 5 min | Maryland highways |
| `vdot-cameras.ts` | VDOT 511 | NoVA cameras | 5 min | 418 metro cams, ~45s stills + HLS; Arlington-jurisdiction relays deduped |
| `arlington-cameras.ts` | Arlington County | County cameras | 5 min | 284 ITS cams, HLS-only (in-modal hls.js) |
| `pgc-cameras.ts` | PG County TRIP | County cameras | 5 min | County-owned HLS; CHARTFeed relays deduped |
| `weatherbug-cameras.ts` | WeatherBug | DMV stills | 5 min | 30 cams, stable instacam still URLs |
| `hivis-cameras.ts` | USGS HIVIS | River-gauge cams | 5 min | Both regions; stills at flood gauges, feed timestamps |
| `mdchart-incidents.ts` | MD CHART | Traffic incidents | 1 min | Crashes, closures |
| `dc-crime.ts` | DC Open Data | Crime reports | 15 min | ArcGIS REST |
| `moco-crime.ts` | Montgomery County | Crime reports | 15 min | Regional expansion |
| `pg-crime.ts` | Prince George's County | Crime reports | 15 min | Regional expansion |
| `dc-shotspotter.ts` | DC Open Data | Gunshot alerts | 5 min | ShotSpotter data |
| `dc-traffic.ts` | DC HSEMA | Traffic incidents | 1 min | DC-specific incidents |
| `alertdc.ts` | AlertDC | Major emergencies | 2 min | Fires, hazmat, etc. |
| `nws-weather.ts` | NWS API | Weather alerts | 2 min | Polygons included |
| `current-weather.ts` | Open-Meteo | Current conditions | 5 min | No API key needed |
| `wmata.ts` | WMATA API | Metro alerts | 30 sec | Requires API key |
| `airnow.ts` | AirNow API | Air quality | 30 min | Requires API key |
| `idaho511-events.ts` | Idaho 511 | Live crashes/hazards | 1 min | ITD `Incidents` + `WazeIncidents` layers; complete listing; jams/closure reports dropped |
| `openmhz.ts` | OpenMHz | Scanner calls | 5 min | Archived transmissions |
| `dcfireems-twitter.ts` | Twitter/X | Fire/EMS tweets | 2 min | Optional, $100/mo API |
| `landmark-webcams.ts` | Multiple | Curated webcams | 5 min | 23 cameras (Senate, NPS, FOX5, etc.) |
| `opensky.ts` | OpenSky Network | Aircraft positions | varies | OAuth2 (`OPENSKY_CLIENT_ID`/`SECRET`); gated by per-client `wantsAircraft` preference |
| `news.ts` | RSS feeds | News items | varies | rss-parser |

The frontend SSE client handles event types `incident:new/update/clear`, `camera:update`, `weather:alert/clear/current`, `aqi:update`, `aircraft:update`, `news:update`, `scanner:update`, plus `connected`/`heartbeat`. See `packages/frontend/src/lib/services/sse.ts`.

### Camera Sources (landmark-webcams.ts)

Curated DC webcams (refreshed 2026-07-04 — every entry live-verified):
- **Official (3)**: US Capitol (Senate.gov), Washington Monument (NPS), NPS Air-quality Mall cam (direct still)
- **YouTube 24/7 (4)**: earthTV White House, FOX 5 DC Skyline, Union Station railcam, EarthCam Monument
- **FOX 5 DC (8)**: Wharf, Stacks, Gaithersburg, Rockville, National Harbor, Reston, Loudoun, Prince William Marina
- **EarthCam pages (3)**: Cherry Blossoms, Kennedy Center, MLK Memorial
- **National Zoo HLS (5)**: Panda, Elephant, Lion, Naked Mole-Rat, Ferret (Wowza, CORS *, play in-modal)
- **Seasonal (1)**: BloomCam

**Note:** the old DDOT "DC street cameras" layer was REMOVED (2026-07): the
dataset is a 2021 pole inventory with no feeds, and DC-proper street camera
imagery no longer exists publicly (DDOT viewer dead, TrafficLand keyed).

### API Endpoints
| Endpoint | Description |
|----------|-------------|
| `GET /api/health` | Health check with service status |
| `GET /api/events` | SSE stream (sends a full snapshot on connect, then deltas); `?aircraft=true&region=<id>` seeds the aircraft preference |
| `POST /api/events/preferences` | Update a client's aircraft preference (`{clientId, wantsAircraft, regionId}`) |
| `GET /api/incidents` | Active incidents (filterable) |
| `GET /api/cameras` | All cameras |
| `GET /api/weather` | Weather alerts |
| `GET /api/aqi` | Air quality data |
| `GET /api/news` | News items |
| `GET /api/news/related/:incidentId` | News related to one incident (region-aware) |
| `GET /api/history/summary?region=&days=` | Daily incident counts by type (SQLite history) |
| `GET /api/history/hourly?region=&hours=` | Hourly incident counts by type |

### Environment Variables
Copy `.env.example` to `.env` — it is the authoritative, commented reference.
Highlights:

```env
REGION=dc                 # Default region hint only — the backend serves ALL regions
PUBLIC_REGION=dc          # Initial frontend selection (user choice persists in localStorage)
REDIS_URL=redis://localhost:6379   # Optional — falls back to in-memory cache

# Optional API keys (features degrade gracefully without them)
WMATA_API_KEY=xxx         # developer.wmata.com (free)
AIRNOW_API_KEY=xxx        # airnowapi.org (free)
TWITTER_BEARER_TOKEN=xxx  # developer.twitter.com ($100/mo)
OPENSKY_CLIENT_ID=xxx     # opensky-network.org OAuth2 (4000 credits/day free)
OPENSKY_CLIENT_SECRET=xxx

# PUBLIC_DEFAULT_LAT/LNG are the MD CHART 80km proximity anchor (backend),
# NOT the frontend map center — that comes from $lib/config region presets.

# Frontend -> backend wiring
PUBLIC_API_URL=           # Empty = same-origin. Set for cross-origin deployments.

# Backend CORS
CORS_ORIGINS=             # Comma-separated. Only consulted when NODE_ENV=production.
                          # In dev, CORS is wide-open ('*'). See packages/backend/src/middleware/cors.ts.
```

### Frontend ↔ Backend Wiring
- In dev: Vite proxies `/api/*` to `http://localhost:3000` (see `vite.config.ts`), so `PUBLIC_API_URL` can be empty.
- In production builds: the SSE client reads `import.meta.env.PUBLIC_API_URL` and prefixes every API call with it (`packages/frontend/src/lib/services/sse.ts`). Empty = same-origin.
- Backend CORS in production reads `CORS_ORIGINS` (comma-separated); if unset it falls back to `http://localhost:5173,http://localhost:4173`.

### CI & GitHub Pages Deployment
Two workflows:
- `.github/workflows/ci.yml` — quality gate on every push: backend
  build (tsc), backend tests (vitest run), lint (both workspaces),
  svelte-check, frontend build. Keep it green.
- `.github/workflows/deploy.yml` — deploys the frontend as a static SPA to
  GitHub Pages on push to `master`.

Deploy details:
- Adapter: `@sveltejs/adapter-static` with `fallback: 'index.html'` (SPA mode).
- Root layout (`src/routes/+layout.ts`): `prerender = true`, `ssr = false`.
- Build-time env vars used by the workflow:
  - `BASE_PATH=/Situation-Monitor` — sets SvelteKit `paths.base` so asset URLs are correct under `/<repo>/`. Empty in dev.
  - `PUBLIC_API_URL` — read from the `PUBLIC_API_URL` repo *variable* (Settings → Secrets and variables → Actions → Variables). Unset = empty = the deployed site renders with no live data. To make the public site work, stand up a stable backend hostname (named Cloudflare Tunnel or Tailscale Funnel — never a quick tunnel, those rotate on restart), set the variable, and re-run the deploy workflow. Backend side: run with `NODE_ENV=production` and `CORS_ORIGINS=https://<owner>.github.io`.
- One-time setup: repo Settings → Pages → Source = "GitHub Actions".
- Deployed URL: `https://<owner>.github.io/Situation-Monitor/`.

## Key Patterns

### Adding a New Data Source
1. Create fetcher in `packages/backend/src/fetchers/`
2. Extend `BaseFetcher<T>` class with `fetchFromApi()` method. Throw on
   failures/contract changes (BaseFetcher then serves stale cache and records
   the error) — never swallow errors into an empty-array "success".
3. Normalize data to `Incident`, `Camera`, or custom type. Derive `updatedAt`
   from feed fields (never `now`); keep any fetch-window filter inside the
   source's `expirationMs` in `aggregator.cleanupStaleIncidents`.
4. Register it in the region pack (`regions/dc.ts` / `regions/boise.ts`) in
   the fetcher array matching its scheduling profile. If the feed is a
   complete snapshot, declare `readonly incidentSource = '<source>' as const`
   and add the source to the pack's `sourcesWithCompleteListing`. If the
   records are ongoing situations (work zones, active fires) rather than
   point-in-time events, set `metadata.ongoing = true` so the frontend's
   event-time filter doesn't hide them.
5. Update `packages/backend/src/types/index.ts` if new types needed (add the
   `DataSource` value)
6. Add SSE event handler in frontend `src/lib/services/sse.ts`
7. Add store in `src/lib/stores/` if needed

### Frontend State Management
Svelte stores in `packages/frontend/src/lib/stores/`:
| Store | Purpose |
|-------|---------|
| `incidents.ts` | Map of all incidents, derived stores for active/byType/counts/metroDelays |
| `cameras.ts` | Map of cameras |
| `weather.ts` | Weather alerts, current weather, AQI |
| `filters.ts` | UI filter state (types, severity, time, layers, heatmap toggle) |
| `location.ts` | Map state, user location, dark mode, sidebar open, search location |

### Map Features
- **Marker Clustering**: Uses leaflet.markercluster for performance
- **Crime Heatmap**: Toggle via `filters.showCrimeHeatmap`, uses leaflet.heat
- **Weather Polygons**: NWS alert zones rendered as Leaflet polygons
- **Search Markers**: Temporary markers from address search (auto-remove after 10s)

### PulsePoint Scraping
The PulsePoint fetcher uses Playwright to:
1. Launch headless Chromium
2. Navigate to PulsePoint web app
3. Add DC FEMS agency (EMS1205)
4. Extract incident data from DOM
5. Parse and normalize to Incident type

**Resource optimization**: Only runs when SSE clients are connected (`sse.getClientCount() > 0`)

### Adding a New Incident Type
The `IncidentType` union is duplicated on both sides — keep them in sync:
1. `packages/backend/src/types/index.ts` and `packages/frontend/src/lib/types/index.ts`
2. Color/name mapping in `packages/frontend/src/lib/utils/format.ts`
3. Filter defaults in `packages/frontend/src/lib/stores/filters.ts` + checkbox in `FilterPanel.svelte`

---
> Source: [ripleyhhunter/Situation-Monitor](https://github.com/ripleyhhunter/Situation-Monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
