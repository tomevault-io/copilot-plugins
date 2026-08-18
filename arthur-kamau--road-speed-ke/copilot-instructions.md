## road-speed-ke

> Open-source database of Kenyan road speed limits and road hazards. Four delivery surfaces: a Go REST API backed by PostgreSQL+PostGIS, a SvelteKit web frontend with Leaflet maps, a Chrome extension that overlays speed zones on Google Maps, and a Kotlin Multiplatform mobile app that includes Android Auto projection (Car App Library) in the same APK. The canonical speed data lives as GeoJSON files in `data/geojson/` — the database is populated from these files. Hazard data (bumps, rumble strips, speed cameras) lives in `data/geojson/hazards/`.

# Kenya Speed Limits — CLAUDE.md

## What This Project Is

Open-source database of Kenyan road speed limits and road hazards. Four delivery surfaces: a Go REST API backed by PostgreSQL+PostGIS, a SvelteKit web frontend with Leaflet maps, a Chrome extension that overlays speed zones on Google Maps, and a Kotlin Multiplatform mobile app that includes Android Auto projection (Car App Library) in the same APK. The canonical speed data lives as GeoJSON files in `data/geojson/` — the database is populated from these files. Hazard data (bumps, rumble strips, speed cameras) lives in `data/geojson/hazards/`.

The KMP mobile app lives in a separate sibling repo, [speed-ke-mobile](https://github.com/Arthur-Kamau/speed-ke-mobile), not in this repo. It consumes the same deployed API. `kmp/` and `android-auto/` are gitignored here in case a local checkout is placed alongside this repo for convenience, but they are not tracked or pushed from this repo. (The former standalone `android-auto/` project was merged into the KMP app and deleted.)

## Architecture

```
cmd/api/              → API server entrypoint (gin, port 8080)
cmd/scraper/          → Data scraper / seed loader CLI
internal/api/         → HTTP handlers + router (gin + cors)
internal/db/          → PostgreSQL connection pool (pgx) + spatial queries
internal/models/      → Go structs (RoadSegment, RoadHazard, BBoxQuery, Stats)
internal/scraper/     → Kenya Law scraper (colly) + GeoJSON seed loader
data/geojson/         → Source-of-truth speed limit data (GeoJSON FeatureCollections)
data/geojson/hazards/ → Road hazard data (bumps, rumble strips, speed cameras)
migrations/           → PostgreSQL + PostGIS DDL (golang-migrate format)
frontend/             → SvelteKit 2 + Svelte 5 (runes mode) + Leaflet + TypeScript
extension/            → Chrome Manifest V3 extension
```

Mobile app (separate repo, [speed-ke-mobile](https://github.com/Arthur-Kamau/speed-ke-mobile)):

```
kmp/                  → Kotlin Multiplatform app (Compose, OSMDroid map, Android target)
                        + Android Auto projection (Car App Library 1.4.0 CarAppService)
```

## Commands

### Go backend
```bash
docker compose up -d                    # Start PostgreSQL+PostGIS
migrate -path migrations -database "$DATABASE_URL" up  # Run migrations
go run cmd/scraper/main.go --seed       # Load GeoJSON into database
go run cmd/api/main.go serve            # Start API on :8080
go build ./...                          # Build all packages
go vet ./...                            # Lint
go test ./...                           # Test
make dev                                # All-in-one: db + migrate + seed + serve
```

### Frontend (SvelteKit)
```bash
cd frontend
pnpm install                             # Install deps
pnpm run dev                             # Dev server on :5173 (proxies /api to :8080)
pnpm run build                           # Production build
npx svelte-check --tsconfig ./tsconfig.json  # Type check
```

### Data management
```bash
go run cmd/scraper/main.go --scrape --output data/scraped.json  # Scrape Kenya Law
# Regenerate static fallback after editing GeoJSON:
python3 -c "import json,glob; ... " > frontend/static/speeds.json
```

## Code Conventions

- **Go**: Standard library style. `internal/` for non-exported packages. pgx for Postgres (not database/sql). Gin for HTTP. No ORM.
- **Frontend**: Svelte 5 runes mode (`$state`, `$derived`, `$effect`, `$props`). No Svelte 4 stores or reactive statements. TypeScript strict. Components in `src/lib/components/`, services in `src/lib/services/`, types in `src/lib/types/`.
- **GeoJSON**: Each file is a FeatureCollection. Coordinates are `[longitude, latitude]` (GeoJSON standard). Properties must include: `road_name`, `speed_limit_kmh`, `road_class` (urban|peri_urban|highway|expressway), `direction`, `source`, `verified`, `county`, `last_updated`.
- **road_class** values are constrained by a CHECK in the database: `urban`, `peri_urban`, `highway`, `expressway`.

## Key Design Decisions

- **GeoJSON is source of truth**, not the database. The database is derived via the seed command, which **truncates and fully reloads `road_segments` inside one transaction on every run** (see `internal/scraper/seed.go`) — this is what it means for the DB to be "derived": renamed/edited/removed segments in GeoJSON are reflected exactly, with no stale or duplicate rows left behind from previous deploys. `road_segments` has no other write path, so this is safe. Edit `data/geojson/*.geojson` to change speed data.
- **Frontend works offline** — if the Go API is unreachable, it falls back to `frontend/static/speeds.json` (a bundled copy of all GeoJSON data).
- **Routing/geocoding provider is switchable.** Default is Google (Places Autocomplete + Directions API with alternative routes) when `VITE_GOOGLE_MAPS_API_KEY` is set in `frontend/.env`; falls back automatically to the free stack (Nominatim geocoding + OSRM routing, no key, no billing) if the key is absent, or if `VITE_MAP_PROVIDER=free` is set explicitly. See `frontend/src/lib/services/mapConfig.ts`. Map tiles stay OpenStreetMap/Leaflet either way — only geocoding and routing switch providers.
- **Route-to-speed matching** happens client-side in `frontend/src/lib/services/matcher.ts` — it finds the nearest speed limit segment within 200m of each route point. This means GeoJSON coordinates must actually follow the real road (see `/speed-data` skill's coordinate-verification step) — segments placed more than ~200m off the real alignment will silently never match on a live route, even though they show up fine in bbox queries.
- **Nairobi Expressway** is 80 km/h (not the dual carriageway default of 110), per NTSA directive. This is a deliberate exception.
- **Speed limits come from Kenya Traffic Act Cap 403 Section 42 and Legal Notice 62/1975**. Legal sources are documented in `data/LEGAL_SOURCES.md`.
- **Feedback email**: kamaukenn11@gmail.com — shown in the webapp sidebar footer and feedback section.

## Database

PostgreSQL 16 + PostGIS 3.4. Connection via `DATABASE_URL` env var.
Default dev credentials: `speed:speed_dev@localhost:5432/speed_limits` (see docker-compose.yml).

Single table: `road_segments` with a `geometry GEOMETRY(LineString, 4326)` column and a GIST spatial index. Queries use `ST_MakeEnvelope` for bbox and `ST_DWithin` for route proximity.

## Environment Variables

```
DATABASE_URL=postgres://speed:speed_dev@localhost:5432/speed_limits?sslmode=disable
PORT=8080
GIN_MODE=debug
GOOGLE_CLIENT_ID=      # OAuth Web Client ID (auth + admin)
ADMIN_PATH_SLUG=       # secret admin URL slug (openssl rand -hex 16); unset = admin disabled
ADMIN_EMAILS=          # comma-separated admin Google emails
```

Copy `.env.example` to `.env` for local development.

## Hazard Types

- `bump` — physical speed bump
- `rumble_strip` — transverse rumble strips
- `speed_camera` — fixed or frequent mobile NTSA camera position
- `pothole` — severe pothole reported by driver

Hazards are stored in `road_hazards` table (Point geometry) and served at `GET /api/v1/hazards?bbox=...`.
Community-submitted hazards via `POST /api/v1/hazards` go in unverified state.
Community-submitted speed observations via `POST /api/v1/speeds/report` go into `speed_reports` (reviewed before promoting to `road_segments`).

## Authentication

Google Sign-In via ID token verification. The backend verifies tokens against Google's `oauth2.googleapis.com/tokeninfo` endpoint — no JWT library needed.

- `POST /api/v1/auth/google` — accepts `{ "id_token": "..." }`, verifies with Google, upserts user in `users` table, returns user profile.
- `GET /api/v1/auth/me` — returns current user (requires `Authorization: Bearer <id_token>` header).
- `OptionalAuth` middleware on `POST /hazards` and `POST /speeds/report` — if a valid auth header is present, links the report to `user_id`; if absent, the report is anonymous (backward compatible).
- `RequireAuth` middleware on `GET /auth/me`.
- `users` table (migration 003): `id`, `google_id` (unique), `email`, `name`, `picture_url`, `created_at`.
- `road_hazards` and `speed_reports` tables have an optional `user_id BIGINT REFERENCES users(id)` column (also migration 003).
- Auth code lives in `internal/api/auth.go` (handler), `internal/api/middleware.go` (middleware), `internal/db/users.go` (queries).
- Requires `GOOGLE_CLIENT_ID` env var (the OAuth Web Client ID from Google Cloud Console) for audience validation.

## Admin Panel

Review queue for community submissions, hidden behind a secret URL (see `docs/PLAN-voice-and-admin.md`).

- API routes live under `/api/v1/{ADMIN_PATH_SLUG}/admin/*` and only exist when `ADMIN_PATH_SLUG` is set. `RequireAdmin` middleware (in `internal/api/middleware.go`) is the real gate: Google ID token + verified email in `ADMIN_EMAILS` (comma-separated). Every auth failure returns a plain 404 so probing is indistinguishable from a missing route.
- Endpoints: `GET /pending/hazards`, `GET /pending/speeds`, `POST /hazards/:id/verify`, `DELETE /hazards/:id`, `POST /speeds/:id/approve`, `POST /speeds/:id/reject`, `GET /stats`. Handlers in `internal/api/admin.go`, queries in `internal/db/admin.go`.
- Migration 004: `speed_reports.status` (pending/approved/rejected) + `reviewed_by`/`reviewed_at` on both report tables.
- **Approving a speed report never writes `road_segments`** (the seed truncate-reloads it from GeoJSON). Approved reports are a to-do list for manual `data/geojson/` edits.
- Frontend: `frontend/src/routes/[slug]/admin/` — not prerendered; adapter-static builds a `404.html` SPA fallback, so the web server must rewrite unknown paths to `/404.html`. Frontend needs `VITE_GOOGLE_CLIENT_ID` for the sign-in button.

## KMP Mobile App (with Android Auto)

Lives in the separate [speed-ke-mobile](https://github.com/Arthur-Kamau/speed-ke-mobile) repo — clone it as a sibling directory (e.g. `../speed-ke-mobile`) to work on it. It is not part of this repo's git history; see `.gitignore` for the local exclusions.

**One app** — Compose Multiplatform, Android target. Open in Android Studio: `File → Open → kmp/`.
Features: OSMDroid map with speed overlays + hazard markers, GPS proximity alerts (same 2km/1km logic),
add speed limit report, add hazard report (both POST to deployed API), Google Sign-In, offline route cache.

The same APK also exposes Android Auto projection: `composeApp` androidMain has a `car/` package
(`SpeedCarAppService` + `SpeedSession` + `SpeedLimitScreen`, Car App Library 1.4.0, NAVIGATION category)
driven by the existing `ProximityMonitorService` broadcasts. Test with the
[Desktop Head Unit (DHU)](https://developer.android.com/training/cars/testing/dhu).

## Deployment

CI/CD (`.github/workflows/deploy.yml`, on push to `main`) is the primary path. `deploy.sh` is
gitignored — copy `deploy.example.sh` to `deploy.sh` for manual deploys. Never commit `deploy.sh`,
it contains server credentials. Server provisioning (Caddyfile, systemd unit, `server-setup.sh`)
lives in the separate [speed-ke-deploy](https://github.com/Arthur-Kamau/speed-ke-deploy) repo.

**Required GitHub secrets** — `DEPLOY_SSH_KEY`, `DEPLOY_HOST`, `DEPLOY_USER`, `DATABASE_URL`,
`GOOGLE_MAPS_API_KEY`, `GOOGLE_CLIENT_ID`, `ADMIN_PATH_SLUG`, `ADMIN_EMAILS`. Set them with
`github-secrets-setup.sh` in the deploy repo.

Two of those are needed in **both** places, which is easy to miss:

- `GOOGLE_CLIENT_ID` is consumed twice — baked into the frontend bundle at build time as
  `VITE_GOOGLE_CLIENT_ID` (the admin panel's sign-in button) *and* written to the server `.env` as
  `GOOGLE_CLIENT_ID` (so the API can validate ID token audiences). Setting only one leaves admin
  half-broken in a way that looks like an auth bug.
- `ADMIN_PATH_SLUG` / `ADMIN_EMAILS` are server-side only, merged into `.env` by the deploy. While
  `ADMIN_PATH_SLUG` is empty the admin routes are never registered and everything 404s — which is
  indistinguishable from a misconfigured slug, so check the server `.env` before debugging further.

The deploy **merges** these keys into the server `.env` key-by-key rather than rewriting the file,
so `DATABASE_URL` and anything hand-added on the box survive. Empty secrets are skipped, so an
unset secret never blanks a working key.

## Speed Limit Rules (Kenya)

- 50 km/h — all built-up areas (towns, cities, trading centres)
- 30 km/h — school zones, health facilities, playgrounds
- 110 km/h — dual carriageway highways (private vehicles)
- 100 km/h — single carriageway highways (private vehicles)
- 80 km/h — all PSVs/commercial vehicles on any road
- 65 km/h — vehicles towing trailers

---
> Source: [Arthur-Kamau/road-speed-ke](https://github.com/Arthur-Kamau/road-speed-ke) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
