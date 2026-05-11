## smart-city-thailand-monitor

> A **72-inch wall display** dashboard for the Bangkok Governor's Integrated Operations Center. Shows real-time urban data: traffic, flooding, air quality, citizen complaints, news, satellite imagery, CCTV feeds, and market signals. Designed to make a Governor say "this is the best command center I've ever seen."

# Bangkok Governor's IOC — Smart City Thailand Monitor

## What This Project Is

A **72-inch wall display** dashboard for the Bangkok Governor's Integrated Operations Center. Shows real-time urban data: traffic, flooding, air quality, citizen complaints, news, satellite imagery, CCTV feeds, and market signals. Designed to make a Governor say "this is the best command center I've ever seen."

This is NOT a generic smart city template. It is a production-grade, data-dense, dark-themed operational dashboard that pulls from 30+ live data sources and renders everything on a single screen with zero wasted pixels.

## Architecture

### Monorepo Structure
```
apps/
  web/          # React 18 + Vite 5 + Leaflet (frontend)
  api/          # Fastify 5 (backend, 25+ adapters, sync engine)
packages/
  shared/       # TypeScript types, seed data, map layers
```

### Tech Stack
- **Frontend:** React 18, Vite 5, Leaflet, TanStack Query
- **Backend:** Fastify 5, TypeScript
- **Maps:** Leaflet + Mapbox/ESRI/CartoDB tiles, Overpass API for road GeoJSON
- **Data:** 29 adapters pulling from CKAN portals, RSS feeds, REST APIs, STAC catalogs
- **No database** — all state is in-memory with periodic snapshot to `tmp/api-state.json`

### Deployment
| Service | Platform | URL |
|---------|----------|-----|
| Frontend (static) | Cloudflare Pages | `bangkok-ioc.pages.dev` |
| API | Render | `smart-city-monitor-api.onrender.com` |

- **Cloudflare Pages** builds from `main` branch using `netlify.toml`
- **Render** builds from `main` branch using `render.yaml`
- Push to `main` = auto-deploy both services
- Working branch: `codex/award-alignment`

### Build Commands
```bash
npm run build -w packages/shared    # Must build first (shared types)
npm run build -w apps/api           # Backend
npm run build -w apps/web           # Frontend (includes tsc check)
```

### Dev Servers
```bash
npm run dev -w apps/api             # localhost:4000
npm run dev -w apps/web             # localhost:5173
```

Frontend uses `VITE_API_BASE_URL` env var. Locally it defaults to `""` (same-origin proxy or direct). In production it points to the Render API.

## Design Philosophy — CRITICAL

Read the parent `~/Projects/CLAUDE.md` for the full manifesto. Here are the rules that matter most for this project:

### Visual Rules (HARD REQUIREMENTS)
- **border-radius: 0px** everywhere. No rounded corners. The only exceptions: small status dots (50% for circles), input fields (functional 2px max).
- **No backdrop-filter blur.** No glassmorphism. Flat, honest panels.
- **No hover animations.** No bouncy transitions. Subtle opacity changes only.
- **No default blue** (#3B82F6). Use project palette: dark bg `rgb(10, 14, 20)`, text `rgb(232, 237, 243)`, accent colors from data meaning (red=danger, green=safe, amber=warning, cyan=data).
- **No boxed SaaS grid layouts.** Asymmetric, editorial, information-dense.
- **Dark theme.** This is an operations center, not a marketing site.

### Information Density
- Every pixel must convey meaning. No decorative spacing.
- The sidebar should pack 12-14 sections into ~800px without scrolling.
- CSS gaps: 0.2-0.3rem between sections. Padding: 0.25-0.4rem inside cards.
- Compress aggressively: single-line items, inline pills, 2-column micro-grids.
- Show MORE data in LESS space. If a section has 3 items and the data has 8, show 8 in a tighter format.

### Typography
- Body: Inter. Headings: Manrope. Thai: Noto Sans Thai.
- Weight contrast over size contrast. Use font-weight 300 for atmosphere, 600 for emphasis.
- Eyebrow labels: 0.6rem, uppercase, letter-spacing 0.04em, opacity 0.6.

## Data Architecture

### Adapter Pattern
Every external data source has an adapter in `apps/api/src/adapters/`. Pattern:

```typescript
export async function syncSourceName(): Promise<AdapterSyncResult> {
  const payload = await fetchJsonOrNull<ResponseType>(config.endpoint);
  if (!payload) return buildResult({ sourceId, status: "stale", ... });
  // Transform to normalized types
  return buildResult({ sourceId, status: "live", newsItems, projectRecords, ... });
}
```

All adapters return `AdapterSyncResult` which can include:
- `newsItems: NewsItem[]` — syndicated content
- `projectRecords: ProjectRecord[]` — tracked initiatives
- `mapFeatureCollections: MapFeatureCollection[]` — GeoJSON for the map
- `mediaFeeds: MediaFeedItem[]` — video/stream links
- Various snapshot patches: `resiliencePatch`, `traffyFonduePatch`, `floodStatusPatch`, `trafficCongestionPatch`, `socialListeningPatch`, `marketSnapshotPatch`

### Sync Engine (`apps/api/src/services/sync.ts`)
- **Ops sync** (every 60s): traffic, air quality, weather, flood, Traffy Fondue
- **Full sync** (every 180s): all of the above + news, catalogs, satellites, markets, CKAN portals

### CKAN Adapters
Three Thai government CKAN portals share helpers in `ckanHelpers.ts`:
- **BMA City Data Portal** (citydataportal.bangkok.go.th) — Bangkok municipal data
- **TAT Tourism** (datacatalog.tat.or.th) — visitor statistics
- **TCEB MICE** (opendata.tceb.or.th) — convention/exhibition data

### Key Bangkok Sources
| Source | Adapter | Data |
|--------|---------|------|
| Traffy Fondue | `traffyFondueAdapter.ts` | Citizen complaints (311) |
| BMA Flood | `bmaFloodAdapter.ts` | Flood level, pumps, rainfall |
| iTIC / Longdo | `iticAdapter.ts` | Traffic events, cameras, congestion index |
| Air4Thai | `air4thaiAdapter.ts` | Official Thai AQI readings |
| Open-Meteo | `openMeteoWeatherAdapter.ts` | Weather forecast |
| Overpass API | `overpassAdapter.ts` | Bangkok highways, arterials, waterways (GeoJSON) |
| BMA GIS | `bmaGisAdapter.ts` | ArcGIS infrastructure layers |
| CartoDB | (tile layer in InteractiveMap.tsx) | Road name labels overlay |

### Map Layers
Managed in `packages/shared/src/mockData.ts` (`mapLayers` array) and rendered by `apps/web/src/InteractiveMap.tsx`. Layer toggles live in `App.tsx` via `operationalLayerToggleIds`.

Adding a new map layer:
1. Add `MapLayerConfig` entry to `mapLayers` in `mockData.ts`
2. Add layer ID to `operationalLayerToggleIds` in `App.tsx`
3. Add color entry in `InteractiveMap.tsx` layer color map
4. Add `useQuery` call in `App.tsx` if it needs API data
5. Add weight/style rules in `renderFeatureCollections()` if needed

### Frontend Data Flow
```
App.tsx → useQuery({ queryFn: fetchFromApi("/api/endpoint", fallbackSeed) })
        → TanStack Query caches + polls at LIVE_POLL_INTERVAL_MS (180s)
        → If API unreachable, falls back to seed data from mockData.ts
        → Renders in sidebar sections within `.overview-shell`
```

## Key Files

| File | Lines | What It Does |
|------|-------|-------------|
| `apps/web/src/App.tsx` | ~5500 | Everything: routing, data queries, sidebar, all UI sections |
| `apps/web/src/styles.css` | ~7500 | All CSS including dark theme, density rules, responsive |
| `apps/web/src/InteractiveMap.tsx` | ~500 | Leaflet map, tile layers, GeoJSON rendering, layer toggles |
| `packages/shared/src/mockData.ts` | ~5000 | Seed data, map layers, source records, city configs |
| `packages/shared/src/types.ts` | ~600 | All TypeScript interfaces |
| `apps/api/src/server.ts` | ~400 | All Fastify routes |
| `apps/api/src/config.ts` | ~120 | All endpoint URLs (env-var backed) |
| `apps/api/src/services/sync.ts` | ~90 | Adapter orchestration |
| `apps/api/src/data/store.ts` | ~500 | In-memory state, snapshot merge logic |

## Common Patterns

### Adding a New Data Source
1. Create `apps/api/src/adapters/myAdapter.ts` (follow existing pattern)
2. Add endpoint to `apps/api/src/config.ts`
3. Register in `apps/api/src/services/sync.ts` (`fullSyncTasks` for 180s, `opsSyncTasks` for 60s)
4. Add source record to `packages/shared/src/mockData.ts` (`sources` array)
5. If it needs a dedicated API route, add to `apps/api/src/server.ts`
6. For frontend: add `useQuery` call in `App.tsx`, render in sidebar

### Adding a New Sidebar Section
1. Compute data in `App.tsx` (derive from existing queries or add new one)
2. Add JSX section inside the `overview-shell` div (around line 4934-5253)
3. Add CSS in `styles.css` (keep gaps tight: 0.2-0.3rem)
4. Section pattern:
```tsx
<section className="card overview-card my-section">
  <div className="card-header">
    <span className="eyebrow">{lang === "th" ? "ชื่อ" : "Name"}</span>
    <span className="status-pill">{count}</span>
  </div>
  <div className="overview-inline-list">
    {items.map(item => (
      <div key={item.id} className="data-item compact">
        <strong>{item.label}</strong>
        <small>{item.value}</small>
      </div>
    ))}
  </div>
</section>
```

### Bangkok-Only Features
Some features only appear when `city === "bangkok"`:
- Traffy Fondue complaints
- Flood status panel
- Traffic corridors
- Overpass road/waterway layers

Guard with: `{city === "bangkok" && ( <section>...</section> )}`

### Overpass API Layers
Bangkok highways, arterials, and waterways use Overpass QL queries cached for 24h. The `bangkokOnlyLayers` set in `App.tsx` bypasses the `featureMatchesCity` filter since Overpass features use `properties.citySlug` instead of `properties.city`.

## Environment Variables

### Frontend (VITE_*)
| Var | Default | Purpose |
|-----|---------|---------|
| `VITE_API_BASE_URL` | `""` | API server URL |
| `VITE_SITE_TITLE` | `"Smart City Thailand Command Center"` | Header title |
| `VITE_DEFAULT_CITY` | (none) | Pre-select city on load |
| `VITE_DEFAULT_VIEW` | (none) | Pre-select view (city/national) |
| `VITE_DEFAULT_BASEMAP` | (none) | Pre-select basemap (satellite/atlas) |

### Backend
| Var | Default | Purpose |
|-----|---------|---------|
| `ALLOW_LIVE_FETCH` | `true` | Master switch for all API calls |
| `AUTO_SYNC_ENABLED` | `true` | Enable background sync |
| `OPS_SYNC_INTERVAL_MS` | `60000` | High-frequency ops sync |
| `SYNC_INTERVAL_MS` | `180000` | Full sync cadence |
| `ADMIN_TOKEN` | `"change-me"` | Admin API auth |
| `GEMINI_API_KEY` | `""` | AI assistant (knowledge queries) |
| `COPERNICUS_CLIENT_ID` | `""` | Sentinel Hub satellite imagery |
| `COPERNICUS_CLIENT_SECRET` | `""` | Sentinel Hub satellite imagery |

## What NOT to Do

- **Never use rounded corners.** `border-radius: 0` is sacred.
- **Never add placeholder content.** Every number must be real or from a real API.
- **Never show localhost as a deliverable.** Always push and verify on the live URL.
- **Never add decorative elements.** No icon grids, no gradient cards, no bouncy animations.
- **Never create SaaS-looking UI.** No equal-sized card grids. Asymmetric, editorial.
- **Never use `backdrop-filter: blur()`.** Flat panels, not frosted glass.
- **Never use `tsc --noEmit`.** Always `tsc -p tsconfig.json` (matches CI).
- **Never use curly/smart quotes** in TypeScript strings. ASCII `"` only.
- **Never cache empty API results.** The Overpass adapter had a bug where empty responses from rate limiting got cached for 24h. Always check `features.length > 0` before caching.

## Deployment Checklist

1. `npm run build -w packages/shared && npm run build -w apps/api && npm run build -w apps/web` — all clean
2. Commit on working branch
3. `git checkout main && git merge codex/award-alignment && git push origin main`
4. Wait 2-3 minutes for Cloudflare Pages + Render
5. Verify `bangkok-ioc.pages.dev` loads
6. Check Render dashboard for API health (`/health` endpoint)

## Current State (April 2026)

- **Sidebar:** 12 sections packed into ~757px — Summary Strip, Traffic, Bangkok Today KPIs, Briefing, Social Pulse, Traffy Fondue, Flood Alert, Traffic Corridors, Actions, Activity Feed, News, Media & Trends, Market Ticker, System Status
- **Map layers:** Satellite imagery, Overpass highways/arterials/waterways, road labels, Traffy Fondue, iTIC traffic, CCTV cameras, projects, resilience, Bangkok passages
- **Data sources:** 29 adapters including 3 Thai CKAN portals (BMA, TAT, TCEB)
- **Theme:** Dieter Rams-inspired dark theme, zero border-radius, flat panels

---
> Source: [Nonarkara/smart-city-thailand-monitor](https://github.com/Nonarkara/smart-city-thailand-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
