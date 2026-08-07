## ai-driven-disaster-intelligence

> This repository is a monorepo with a FastAPI backend and a React frontend built with Vite.

# Repository Guidelines

## Project Structure & Module Organization
This repository is a monorepo with a FastAPI backend and a React frontend built with Vite.

- `backend/`: Python API server.
- `backend/main.py`: Unified API for live disaster ingestion, deterministic road-status analysis, smart route comparison, disaster summaries, and OSM road fetching.
- `backend/models/disaster.py`: Shared Pydantic model for normalized disaster records.
- `backend/services/disaster_service.py`: Async disaster aggregation, normalization, caching, deduplication, and external provider integration for ReliefWeb, NASA FIRMS, and OpenWeather.
- `backend/services/analysis_service.py`: Deterministic road status engine using impact radii and Haversine distance.
- `backend/services/routing_service.py`: Standard OSRM routing plus blocked-road-aware smart routing that excludes blocked roads and penalizes risky roads.
- `backend/services/risk_service.py`: Route hazard scoring, sampled route exposure checks, safety-level classification, and route exposure summaries.
- `backend/services/ai_summary_service.py`: LLM-backed disaster insight generation with cached fallback summaries.
- `frontend/`: React app using Zustand and Mapbox GL.
- `frontend/src/components/MapComponent.jsx`: Unified map rendering for roads, blocked-road overlays, clustered disaster markers, interactive disaster popups, and fastest-vs-safest route layers.
- `frontend/src/components/DisasterPanel.jsx`: Live disaster feed, type and confidence filters, and timeline controls.
- `frontend/src/components/RouteComparisonPanel.jsx`: Fastest-vs-safest comparison cards with duration, safety level, and hazard metadata.
- `frontend/src/components/SearchBox.jsx`: Source and destination search with geolocation-aware route input.
- `frontend/src/components/WarningBanner.jsx`: Persistent reliability warning for disaster and road-status uncertainty.
- `frontend/src/store/useAppStore.js`: Shared state for map position, routes, route comparison, live disasters, filters, and timeline state.
- `frontend/src/utils/disasters.js`: Disaster normalization helpers, filter helpers, popup content, and GeoJSON conversion utilities.
- `frontend/src/utils/timeline.js`: Timeline preset generation, normalization, labels, and URL sync helpers.

## Build, Test, and Development Commands

### Backend
- Install dependencies: `cd backend && pip install -r requirements.txt`
- Run dev server: `python main.py`
- Quick syntax check: `python -m compileall .`

### Frontend
- Install dependencies: `cd frontend && npm install`
- Run dev server: `npm run dev`
- Lint: `npm run lint`
- Production build: `npm run build`

## Environment Variables
Backend live disaster aggregation expects:

- `RELIEFWEB_APPNAME`
- `NASA_API_KEY`
- `OPENWEATHER_API_KEY`
- `DISASTER_CACHE_TTL_SECONDS` (optional, defaults to `180`)

Frontend expects:

- `VITE_MAPBOX_TOKEN`
- `VITE_API_BASE` (optional backend base override)

## Coding Style & Naming Conventions

### Python
- Use FastAPI and Pydantic models with explicit type hints.
- Keep private helpers prefixed with `_`.
- Handle partial provider failures gracefully in service modules.
- Prefer extending existing disaster, road-status, and route comparison flows instead of introducing duplicate endpoints.

### JavaScript / React
- Use Zustand for shared app state instead of prop drilling.
- Keep API calls in `frontend/src/services/api.js`.
- Keep disaster and timeline helpers in `frontend/src/utils/`.
- Prefer updating Mapbox GeoJSON sources with `setData` instead of recreating layers.
- Keep route comparison and disaster filter state in the store rather than component-local state.

## Testing Guidelines
There is no formal automated test suite yet. Before submitting changes:

- Run `python -m compileall .` in `backend/`
- Run `npm run lint` in `frontend/`
- Run `npm run build` in `frontend/`
- Manually verify `/disasters` and `/api/disasters`, including `from` and `to` filtering
- Manually confirm disaster markers, cluster expansion, popup content, timeline presets, URL query persistence, legend accuracy, and map interactions
- Manually confirm road overlays: blocked, risky, and safe styling
- Manually confirm route comparison mode: source and destination selection, fastest default selection, sidebar switching, blocked-road-aware safest route behavior, and fallback messaging

## Commit & Pull Request Guidelines
- Use descriptive commit messages such as `Real Disaster Data Integration`.
- Keep commits scoped to the feature or fix being delivered.
- Avoid temporary or placeholder commit messages in published history.

---
> Source: [ramakrishna5201/AI-driven-disaster-intelligence](https://github.com/ramakrishna5201/AI-driven-disaster-intelligence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
