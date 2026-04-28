## route-biodiversity

> A web app that shows biodiversity (species, observations) along a running/cycling route. Users connect their Strava account or upload a GPX file, and the app queries iNaturalist for species observed near the route, displays them on an interactive map, and shows indigenous territory information from Native Land Digital.

# CLAUDE.md — Route to Biodiversity

## Project overview

A web app that shows biodiversity (species, observations) along a running/cycling route. Users connect their Strava account or upload a GPX file, and the app queries iNaturalist for species observed near the route, displays them on an interactive map, and shows indigenous territory information from Native Land Digital.

**Live site:** https://bioroute.pedrolab.org (deployed on Railway)

## Tech stack

- **Backend:** Python 3.11+ / FastAPI (`app/main.py`)
- **Frontend:** Vanilla JS + Tailwind CSS (CDN) + Leaflet.js — single-page app served as static files
- **Deployment:** Docker → Railway (auto-deploys from `master` branch)
- **No database** — sessions stored in signed cookies, activity cache is in-memory

## Architecture

```
app/
├── main.py                  # FastAPI app, middleware, static mount
├── config.py                # Pydantic Settings (env vars)
├── routers/
│   ├── auth.py              # Strava OAuth flow (login, callback, logout, status)
│   └── api.py               # REST endpoints (activities, species, observations, territories, GBIF, eBird)
├── services/
│   ├── strava.py            # Strava API client, token refresh, activity caching
│   ├── biodiversity.py      # iNaturalist, Native Land, BigDataCloud, GBIF APIs, convex hull
│   └── ebird.py             # eBird API client (recent + notable observations)
└── static/
    ├── index.html           # SPA shell (3-panel layout: sidebar, center, map)
    ├── styles.css            # Custom CSS (desktop 3-panel + mobile responsive)
    └── app.js               # All client-side logic (iNat, GBIF merge, eBird rendering)
```

## Key env vars

| Variable | Purpose |
|---|---|
| `STRAVA_CLIENT_ID` | Strava API app ID |
| `STRAVA_CLIENT_SECRET` | Strava API secret |
| `BASE_URL` | App URL for OAuth redirects (e.g. `https://bioroute.pedrolab.org`) |
| `SESSION_SECRET` | Session cookie signing key |
| `NATIVE_LAND_API_KEY` | Native Land Digital API key |
| `EBIRD_API_KEY` | eBird / Cornell Lab API key |

Local dev uses `.env` file. Production uses Railway env vars.

## Running locally

```bash
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```

Then open http://localhost:8000

## Key behaviors

- **Strava activity cache:** On login (and every page load), the last 6 months of activities are fetched from Strava in the background and cached in-memory. Search queries hit this cache for instant results.
- **Convex hull filtering:** Route coordinates are used to compute a convex hull (~200m buffer). Species and observations are filtered to only include those within the hull, preventing irrelevant results from wide bounding boxes.
- **Species fetching:** When a hull is available, species are aggregated from hull-filtered observations (not from `species_counts` API) to ensure accuracy.
- **Token refresh:** Strava tokens are automatically refreshed when expired via `get_valid_token()` in `auth.py`.
- **GBIF data source toggle:** Users can enable GBIF as a secondary data source in the sidebar. GBIF species are merged with iNaturalist data, deduplicated by scientific name. Species cards show source badges (iNat, GBIF, or both).
- **eBird hybrid time ranges:** Recent sightings (7/14/30 days) use the eBird API directly via route sample points. "All time" queries GBIF's eBird dataset (`datasetKey=4fa7b334-...`) for historical data. Note: GBIF's eBird data has a ~2 year lag, so intermediate ranges (6mo, 1yr) aren't offered.

## Frontend layout

- **Desktop:** 3-panel — sidebar (220px, route input + activities + taxa filters + data sources + eBird), center (filters bar + species grid + eBird list), map (50%)
- **Mobile:** After route loaded, sidebar collapses into a "Fetch route" dropdown button. Taxa filters appear as wrapping pills in the filters bar. Data Sources and eBird sidebar sections are hidden on mobile. Map is always visible at 35vh. Species grid is 2 columns.

## External APIs

- **Strava API** — OAuth2 auth, activity list, activity streams (lat/lng)
- **iNaturalist API** — `/v1/observations/species_counts` and `/v1/observations` for species and observation data
- **GBIF API** — `/v1/occurrence/search` for global biodiversity occurrences; also used for historical eBird data via dataset key filtering
- **eBird API** — `/v2/data/obs/geo/recent` and `/v2/data/obs/geo/recent/notable` for recent bird sightings (requires API key)
- **Native Land Digital API** — Indigenous territory lookup (requires API key)
- **BigDataCloud** — Reverse geocoding for country identification

## Deployment

Railway auto-deploys on push to `master`. Uses `Dockerfile` for builds.

```bash
git push origin master
```

To set Railway env vars:
```bash
railway variables set KEY=value
```

## API endpoints

| Endpoint | Method | Description |
|---|---|---|
| `/api/species` | POST | iNaturalist species by taxa (bbox, month, hull) |
| `/api/observations` | POST | iNaturalist observations (bbox, month, taxon_id, hull) |
| `/api/territories` | POST | Native Land territories + country (bbox) |
| `/api/gbif/species` | POST | GBIF species by taxa (bbox, month, hull) |
| `/api/ebird/recent` | POST | eBird recent sightings (coords, dist_km, back_days) |
| `/api/ebird/notable` | POST | eBird notable sightings (coords, dist_km, back_days) |
| `/api/ebird/historical` | POST | eBird all-time historical via GBIF (bbox, hull) |

## Common tasks

- **Add a new taxa group:** Add to `TAXA_LIST` in `biodiversity.py`, add icon in `TAXA_ICON_CLASSES` and label in `TAXA_LABELS` in `app.js`
- **Change hull buffer:** Modify `buffer_deg` parameter in `convex_hull()` in `biodiversity.py` (0.002 ≈ 200m)
- **Icons:** Using Font Awesome 6 (CDN). Taxa icons defined in `TAXA_ICON_CLASSES` in `app.js`
- **Add a new data source:** Add toggle in `#source-toggles` (index.html), handle in `initSourceToggles()` (app.js), create backend service + endpoint

## Style conventions

- Minimal, clean aesthetic (Vercel/Notion-inspired)
- Font: Inter
- Colors: grays (#111827 primary, #9ca3af secondary, #f3f4f6 borders)
- No emoji in code or UI unless explicitly requested

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pinheirochagas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
