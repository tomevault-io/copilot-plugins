## good-ice-map

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Always use the Playwright MCP to check your work.

When presented with a login screen, notify the user and the user will authenticate in the browser.

## Commands

- **Dev server:** `composer dev` — runs PHP server, queue worker, log tail (pail), and Vite concurrently
- **Build:** `npm run build`
- **Setup (fresh install):** `composer setup` — installs deps, generates key, runs migrations, builds frontend
- **Tests:** `composer test` — clears config cache then runs PHPUnit
- **Single test:** `php artisan test --filter=TestClassName` or `php artisan test tests/Feature/SomeTest.php`
- **Linting:** `./vendor/bin/pint` (Laravel Pint, PSR-12 style)
- **Migrations:** `php artisan migrate`
- **Tinker:** `php artisan tinker`

## Architecture

Laravel 12 app for discovering and rating "good ice" locations on a map. Brutalist design aesthetic.

### Backend
- Standard Laravel MVC: controllers in `app/Http/Controllers/`, models in `app/Models/`, services in `app/Services/`
- **LocationController** — main CRUD (index renders map homepage, show/create/store/destroy)
- **RatingController** — rating submission (uses `updateOrCreate` for one-rating-per-user)
- Auth via Laravel Breeze (session-based)
- SQLite database (dev), configured in `.env`
- Image storage on Cloudflare R2 (S3-compatible) via `league/flysystem-aws-s3-v3`

### Frontend
- Blade templates in `resources/views/` — no SPA, server-rendered with Alpine.js for reactivity
- Leaflet.js for the map (`resources/views/locations/index.blade.php`)
- Vite + Tailwind CSS (entry: `resources/css/app.css`, `resources/js/app.js`)
- Rough.js available for sketchy graphics

### Key Routes
- `GET /` — map homepage (public), fetches markers from `GET /api/locations`
- `GET /locations/{id}` — location detail with ratings
- `GET /locations/create` — submit form (auth required), supports Google Maps link parsing or manual entry
- `GET /dashboard` — user's submitted locations (auth required)
- `POST /locations/{id}/rate` — submit rating (auth required)

### External APIs
- **Google Places/Maps APIs** — location search, place details, geocoding (requires `GOOGLE_PLACES_API_KEY`)
- API helpers in `app/Services/` and routes: `/api/expand-url`, `/api/search-place`, `/api/fetch-place-details`

### Database Models
- **Location** — has many LocationImage and Rating, belongs to User (submitted_by), uses SoftDeletes
- **Rating** — unique constraint on (location_id, user_id), 1-5 integer scale
- **LocationImage** — stores R2 path, has `url` accessor for generating storage URLs

## Design System

Brutalist aesthetic — monospace fonts, thick black borders (3-5px), harsh drop shadows (`shadow-brutal`), purple accent (#9333ea), high contrast black/white. Custom Tailwind config in `tailwind.config.js` defines these utilities.

---
> Source: [kkoppenhaver/good-ice-map](https://github.com/kkoppenhaver/good-ice-map) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
