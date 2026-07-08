## music-cat

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal music catalog app for browsing, rating, tagging, and randomly picking albums. The core feature is the "Surprise Me" random album picker with filters. Google Sheets serves as the persistent data store (read and write); H2 is used as a runtime cache/database. The music library data was scanned once from an external drive via a Python script — the scanner is a one-time prerequisite, not part of the running application. **The app is deployed and live** on Google Cloud Run, scaled to zero when idle, with Google Sheets as the persistent source of truth.

## Current State

All tasks (0–18) from `task-list.md` are complete. The app is deployed to Cloud Run.

**Done:**
- `music_scanner.py` — One-time Python scanner that produced `catalog.json` (prerequisite, already run)
- `catalog.json` — Scanned library (~176 artists, ~2830 albums, ~31K tracks across 7 genres) at project root
- `plan.md` — Full architecture and design document
- `task-list.md` — Sequential implementation tasks (Tasks 0-10)
- **Backend skeleton** — Spring Boot app with Gradle build, Flyway migration, H2 database, application.yml
- **Domain entities** — ArtistEntity, AlbumEntity, SongEntity, TagEntity with JPA mappings
- **Repositories** — ArtistRepository, AlbumRepository (with EntityGraph), SongRepository, TagRepository
- **Catalog import** — CatalogImportService (JSON → DB), CatalogAutoImporter (auto-imports on first startup if DB empty), POST /api/catalog/import endpoint
- **Catalog DTOs** — Java records in `dto.catalog` package: Catalog, Genre, Artist, Album, Stats, ImportResult
- **Artist CRUD API** — ArtistService, ArtistController (full CRUD + favorite toggle + tag management)
- **Album CRUD API** — AlbumService, AlbumController (full CRUD + grade + favorite + tags + rich filtering via AlbumFilterParams)
- **Tag CRUD API** — TagService, TagController (list, create, delete)
- **DTOs** — ArtistDto, ArtistCreateDto, ArtistUpdateDto, AlbumDto, AlbumSummaryDto, AlbumCreateDto, AlbumUpdateDto, AlbumFilterParams, GradeDto, SongDto, TagDto, TagCreateDto
- **Exception handling** — NotFoundException, NoMatchException, GlobalExceptionHandler with @ControllerAdvice (404, 400 validation, 500)
- **Browse API** — BrowseService, BrowseController (genres with counts, artists by genre, albums by artist, tag stats, favorites, full stats with grade distribution)
- **Random Album API** — RandomPickService, RandomController (single random album, multiple random albums, all album filters supported)
- **JPA Specifications** — AlbumSpecs extracted as reusable static utility class (artistGenreEquals, byArtist, hasAnyTag, gradeGte, isFavorite, isUnrated, etc.)
- **Browse DTOs** — BrowseGenreDto, BrowseTagDto, BrowseStatsDto, BrowseFavoritesDto

- **Frontend shell** — React 19 + TypeScript + Vite 7, TanStack Query v5, React Router 7, Tailwind CSS 4, Lucide icons; Vite proxy to :8080 in dev, builds to backend static resources
- **Frontend structure** — `src/types/index.ts` (all TS interfaces), `src/api/client.ts` (typed axios calls), `src/hooks/` (useArtists, useAlbums, useBrowse, useRandomAlbum), `src/components/Layout.tsx + Sidebar.tsx`, `src/pages/DashboardPage.tsx` (live stats)
- **Shared components** — StarRating (clickable 1-5 stars), FavoriteToggle (heart icon), TagBadge (pill with remove), FilterBar (genre/grade/favorite/tag filters), AlbumCard (summary card), ArtistCard (artist card)
- **Browse page** — Genre grid with inline accordion expansion showing artists, drill-down to artist detail
- **Artist pages** — ArtistListPage (filtered grid of ArtistCards), ArtistDetailPage (header + favorite + tags + album grid)
- **Album pages** — AlbumListPage (FilterBar + AlbumCard grid), AlbumDetailPage (header + star rating + favorite + tags + song table)

- **Random Pick page** — "Surprise Me" button with genre/grade/favorite filters, shows full album detail with inline rating/tagging, "Roll Again" flow
- **Favorites page** — Favorite artists and albums grids with empty state, inline unfavorite toggles
- **Tags page** — Tag cloud with sort (name/usage), create/delete tags, click tag to show associated artists and albums

**Deployment phase (Tasks 9-18), all done:**
- **Google Sheets sync** — `GoogleSheetsClient`/`SheetMapper` (chunked writes via `values.append` for rows beyond the first, since `values.update` never grows a sheet's grid past its current row count; 429 retry with backoff); write path pushes Artists+Albums synchronously after every mutating commit (Songs only on structural changes) via `SheetSyncListener`; read path restores the DB from Sheets on an empty boot, or seeds from `catalog.json` and pushes if the sheet is blank; `POST /api/catalog/sync/push`, `POST /api/catalog/sync/pull`, `GET /api/catalog/sync/status`
- **Sync safety** — event-driven pushes start suspended and only resume once the DB provably mirrors the sheet (clean restore/pull/push); any push failure re-suspends (a non-atomic write can leave a tab partial); malformed/duplicate/orphaned sheet rows are skipped with warnings that keep pushes suspended; artists are keyed by name only — two same-named artists in different genres must be disambiguated (`catalog.json` has 4: Genesis, Roland Dyens, Manfred Mann, Scorpions, each suffixed `(Genre)` on their second occurrence)
- **Offline export** — `GET /api/catalog/export/json`, `GET /api/catalog/export/csv` (CSV values starting with `=+-@` are prefixed with `'` against formula injection)
- **Single-jar build** — Gradle downloads Node and builds the React app into the boot jar; `SpaForwardingController` forwards React Router routes to `index.html`
- **HTTP Basic auth on every path** — `SecurityConfig`; refuses to start under the `cloud` profile if credentials still equal the checked-in `admin`/`admin` default; `RequireXhrHeaderFilter` requires `X-Requested-With` on state-changing requests (blocks blind cross-site CSRF — Basic auth has no CSRF-token/SameSite-cookie equivalent, browsers auto-attach cached credentials per-origin regardless of the initiating page)
- **Cloud Run deployment** — `application-cloud.yml` (in-memory H2, `lazy-initialization: true`), `Dockerfile` (non-root user), `deploy.sh`; `ReadinessGateFilter` blocks requests until `CatalogAutoImporter` finishes its boot decision (up to 25s) rather than failing fast — Cloud Run only allocates full CPU during active request processing, so a fail-fast 503 starves the boot-time Sheets restore of CPU (observed live: ~3s locally became ~5 minutes on Cloud Run before this fix; blocking one request dropped it back to ~10-15s)
- **Live and verified end-to-end**: persistence across a forced new revision, hand-edit-then-pull, and a Sheets-outage failure mode (app degrades to `catalog.json` data and reports the error via `sync/status` instead of breaking; ratings still succeed locally; recovers via `sync/push` once access is restored)

## Architecture

Monorepo with two modules that build into a single deployable JAR, deployed as one container to Cloud Run:

- **backend/** — Java 25, Spring Boot 4.0.2, Spring Data JPA, H2 (file-persisted locally, in-memory on Cloud Run), Flyway migrations, SpringDoc OpenAPI 3.0.1, Spring Security (HTTP Basic)
- **frontend/** — React 19, TypeScript, Vite 7, TanStack Query v5, React Router 7, Tailwind CSS 4, Lucide icons
- **catalog.json** — Scanner output at project root (initial seed / fallback when Sheets is unreachable)
- **config/** — Google Sheets service account credentials (gitignored locally; mounted from Secret Manager in the cloud profile)
- **deploy.sh** — builds (buildx, linux/amd64), pushes to Artifact Registry, deploys to Cloud Run

Data flow: Google Sheets (persistent source of truth) ↔ App (H2 runtime cache, rebuilt from Sheets on every boot) ↔ REST API ↔ React UI

Boot decision (`CatalogAutoImporter`): DB non-empty → skip; DB empty + Sheets has data → restore from Sheets; DB empty + Sheets blank → seed from `catalog.json` and push; DB empty + Sheets disabled → import `catalog.json` (local dev default). A Sheets outage at any of these points falls back to `catalog.json` without pushing, so a diverged DB can never overwrite the sheet.

## Key Commands

```bash
# Development (two terminals)
cd backend && ./gradlew bootRun          # Backend on :8080
cd frontend && npm run dev               # Frontend on :5173 (proxies /api to :8080)

# Production build (single JAR)
./gradlew bootJar
java -jar backend/build/libs/music-cat-*.jar   # Serves both API and UI on :8080

# Deploy to Cloud Run
MUSIC_CAT_USER=... MUSIC_CAT_PASSWORD=... SHEETS_SPREADSHEET_ID=... ./deploy.sh
```

## Domain Model

```
Artist (1) ---> (N) Album (1) ---> (N) Song
   |  N:M                |  N:M
   +-------> Tag <-------+
```

- Songs accessed through albums only (no direct Song API)
- Tags are shared between artists and albums via join tables
- `grade` (1-5, nullable) and `isFavorite` are on both Artist and Album
- `year` parsed from album folder names during scan; nullable

## API Structure

All endpoints under `/api/`, all requiring HTTP Basic auth (state-changing requests also require an `X-Requested-With` header):
- `/api/artists` — Artist CRUD + favorite toggle + tag management
- `/api/albums` — Album CRUD + grade + favorite + tags; supports rich filtering (genre, minGrade, tags, favorite, unrated)
- `/api/browse/genres`, `/api/browse/tags`, `/api/browse/stats`, `/api/browse/favorites` — Navigation/discovery
- `/api/random/album`, `/api/random/albums` — Random pick with same filters as album list
- `/api/tags` — Tag CRUD
- `/api/catalog/import` — Import from `catalog.json`
- `/api/catalog/export/json`, `/api/catalog/export/csv` — Offline backups
- `/api/catalog/sync/push`, `/api/catalog/sync/pull`, `/api/catalog/sync/status` — Google Sheets sync (503 when Sheets is disabled)

## Scanner (Prerequisite — Already Complete)

`music_scanner.py` is a one-time script that was run against an external drive to produce `catalog.json`. It is not part of the running application. The external drive is no longer needed.

## Backend Packages

Base package: `io.github.alexshamrai`

- `domain/` — JPA entities: ArtistEntity, AlbumEntity, SongEntity, TagEntity
- `repository/` — Spring Data JPA repos with JpaSpecificationExecutor, @EntityGraph for efficient loading
- `specification/` — AlbumSpecs (reusable static JPA Specification methods for album filtering)
- `service/` — CatalogImportService, ArtistService, AlbumService (uses AlbumSpecs), TagService, BrowseService, RandomPickService, CatalogExportService, SheetSyncService, SheetsCatalogReader, SheetsLoadResult, TagNames
- `controller/` — CatalogController, ArtistController, AlbumController, TagController, BrowseController, RandomController
- `dto/` — Artist DTOs, Album DTOs (AlbumDto, AlbumSummaryDto, AlbumCreateDto, AlbumUpdateDto, AlbumFilterParams, GradeDto), SongDto, TagDto, TagCreateDto, ImportResult, BrowseGenreDto, BrowseTagDto, BrowseStatsDto, BrowseFavoritesDto, SyncResultDto, SyncStatusDto
- `dto/catalog/` — Java records mapping catalog.json: Catalog, Genre, Artist, Album, Stats
- `dto/export/` — ExportCatalog, ExportGenre, ExportArtist, ExportAlbum, ExportSong
- `exception/` — NotFoundException, NoMatchException, GlobalExceptionHandler (@ControllerAdvice)
- `startup/` — CatalogAutoImporter (boot decision tree), ReadinessState
- `sheets/` — SheetsClient (interface), GoogleSheetsClient, SheetMapper, SheetSyncListener, SheetsSyncLock, ArtistRow, AlbumRow, SongRow
- `config/` — WebConfig (Genre converter), SpaForwardingController, GoogleSheetsConfig, SheetsProperties, SecurityConfig, RequireXhrHeaderFilter, ReadinessGateFilter, ReadinessGateConfig

## Important Conventions

- **Google Sheets is the persistent store** — the app reads from and writes to Google Sheets; H2 is a runtime database/cache, fully rebuilt from Sheets on every boot (in-memory and wiped on every scale-to-zero under the cloud profile)
- Album filtering uses JPA Specifications (composable via `.and()`)
- Google Sheets integration is `@ConditionalOnProperty` — disabled by default, no errors when credentials missing. `GoogleSheetsClient` resolves its `Sheets` API client via `ObjectProvider`, not direct injection — a missing/bad credentials file must fail at first real API call, not at Spring context refresh (the latter crash-loops the whole app under the cloud profile's `lazy-initialization: true`, since `SheetSyncListener`/`CatalogAutoImporter` are forced eager for event-listener wiring)
- Custom Spring Security filters must use `response.setStatus(...)` + write the body directly — never `response.sendError(...)`. `sendError` triggers a container-level `/error` forward that Spring Security's filter chain also runs on by default, re-entering the same filter and producing a wrong status code on a real server (MockMvc doesn't replicate this, so it only surfaces in a live deployment)
- IDs are not stable across a `sync/pull` — it fully deletes and reimports, and H2 `IDENTITY` columns regenerate. Never hardcode/cache an artist or album ID across a pull; look up by name instead
- A tag with zero artist/album associations has no representation in the 3-tab sheet schema (tags only persist as a comma-joined value on Artist/Album rows) and is silently dropped on the next `sync/pull`. Tags attached to at least one artist or album survive correctly. Found live during Task 18's round-trip test; not fixed since a fix needs a 4th sheet tab, contradicting the "exactly three tabs" spreadsheet design
- Frontend uses Vite proxy in dev; production build outputs to `backend/src/main/resources/static/`
- SPA routing: Spring forwards non-API/non-static paths to `index.html`
- Schema managed by Flyway (`db/migration/V1__init_schema.sql`), JPA set to `validate` mode
- `year` is a reserved word in H2 — must be quoted (`"year"`) in SQL migrations and JPA `@Column(name = "\"year\"")`
- Java 25 (via sdkman), Gradle 9.0 wrapper, Spring Boot 4.0.2, Lombok 1.18.42
- Spring Boot 4.0 modularization: Flyway requires `spring-boot-starter-flyway`, H2 console requires `spring-boot-h2console`

---
> Source: [alexshamrai/music-cat](https://github.com/alexshamrai/music-cat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
