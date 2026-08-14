## arkade5-nettsted

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

The website and download/statistics backend for **Arkade 5**, a Norwegian archive-extraction validation tool (Nasjonalarkivet, formerly Arkivverket). The repo serves two distinct things from one Laravel 12 app:

1. A **public static marketing/download site** (`public/frontend/`, plain HTML + vanilla JS + Leaflet) served at `/`.
2. A **Laravel admin backend** at `/admin` (auth-gated, Blade + Tailwind + Alpine) for viewing download statistics and managing release builds.

Domain language is Norwegian: *utgivelser* = releases, *nedlastinger* = downloads, *nedlastere* = downloaders, *organisasjoner* = organizations, *statistikk* = statistics.

## Commands

This project uses **Laravel Sail** (Docker). The stack is Postgres 17, Redis, Meilisearch, Mailpit, and Selenium (see `docker-compose.yml`). Sail wraps artisan/composer/npm inside the container.

```bash
./vendor/bin/sail up -d            # start the stack
./vendor/bin/sail artisan ...      # run artisan in-container
./vendor/bin/sail composer ...
./vendor/bin/sail npm run dev       # Vite dev server (HMR for admin assets)
./vendor/bin/sail npm run build     # production asset build

# Tests (PHPUnit, NOT Pest despite the plugin allow-list)
./vendor/bin/sail artisan test                          # full suite
./vendor/bin/sail artisan test --filter ProfileTest      # single test class
./vendor/bin/sail artisan test tests/Feature/Auth/AuthenticationTest.php

# Code style
./vendor/bin/sail pint              # Laravel Pint formatter
```

Without Sail, substitute `php artisan`, `composer`, `npm` directly (requires PHP 8.1+ and a local Postgres).

## Database

**There is no `database/migrations/` directory.** Schema is managed as a squashed dump in `database/schema/pgsql-schema.sql` (and a `mysql-schema.sql` variant). `php artisan migrate:fresh --seed` loads the schema dump and then runs `DatabaseSeeder`, which seeds users, downloaders, organizations, releases, and downloads. When changing the schema, update the schema SQL files directly rather than expecting to add a migration.

Note: `.env.example` ships with `DB_CONNECTION=mysql`, but the real `.env` and the Docker stack use **Postgres** (`DB_CONNECTION=pgsql`, host `pgsql`).

## Architecture

### Routing is logic-heavy — most "controllers" are route closures
Business logic lives directly in `routes/web.php` and `routes/api.php` as closures, not in controllers. There are almost no real controllers (only auth, from Breeze, and `ProfileController`). When adding behavior, expect to read and edit the route closures.

- `routes/api.php` — the public-facing integration surface called by the desktop Arkade client:
  - `POST /api/arkade-downloads` (`download.store`): the core flow. Looks up the matching `ArkadeRelease` by `arkadeUI` + version (or latest), streams the build file from `Storage` (`builds/releases/...`), and **records a download** by upserting an `ArkadeDownloader` (by email), optionally upserting an `Organization` (by org number, enriched via `OrganizationInfoService`), and saving an `ArkadeDownload` linking them.
  - `GET /api/arkade-versions` — distinct version numbers, newest first.
  - `GET /api/organization-locations` — orgs with lat/long, for the front-page Leaflet map.
- `routes/web.php` — `/admin/*` (auth + verified): dashboard, `statistikk/*` statistics views, `builds/*` (browse and download release artifacts from `Storage`), and `news-receivers` (downloaders who opted into news). `require __DIR__.'/auth.php'` pulls in Breeze auth routes.

### Models (`app/Models`)
- `ArkadeRelease` — has a **global scope named `public`** that hides releases without `released_at` or with a `dereleased_at`. Admin queries that need all releases call `withoutGlobalScope('public')`.
- `ArkadeDownload` — uses `downloaded_at` as `CREATED_AT` and **disables `UPDATED_AT`**. Belongs to a release, a downloader, and (optionally) an organization.
- `ArkadeDownloader` — keyed by `email`, has `wants_news` flag.
- `Organization` — keyed by `org_number`; `scopeWithAddressLocation` filters to rows that have coordinates.

### OrganizationInfoService (`app/Services`)
Enriches an `Organization` from two external Norwegian APIs: company data from **Brønnøysundregistrene** (`data.brreg.no/enhetsregisteret`) and geocoding from **Geonorge** (`ws.geonorge.no/adresser`). Failures are caught and logged, never fatal to the download flow. The artisan command `db:update-organization-info` (`app/Console/Commands/UpdateOrganizationInfo.php`) re-runs enrichment across all organizations.

### API Resources (`app/Http/Resources`)
JSON shaping for the statistics views and the locations endpoint. Each model has a singular Resource and a `*Collection` paginated wrapper.

### Frontend assets — two separate pipelines
- The **public site** (`public/frontend/`) is pre-built static output committed to the repo; it is *not* produced by this project's Vite build. It calls the `/api` endpoints directly.
- **Vite** (`vite.config.js`) builds only `resources/css/app.css` and `resources/js/app.js` (Tailwind + Alpine) for the **admin** Blade views.

### Auth
Laravel Breeze (Blade stack) + Sanctum. All `/admin` routes require `auth`; the dashboard also requires `verified`.

---
> Source: [Arkitektum/arkade5-nettsted](https://github.com/Arkitektum/arkade5-nettsted) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
