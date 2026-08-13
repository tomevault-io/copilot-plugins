## mkulimaapp-backend

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development (auto-restart on changes)
npm run dev

# Production
npm start

# Run all tests
npm test

# Run a single test file
node --test --experimental-test-isolation=none test/intelligence/maize.fullFlow.test.js
```

## Environment Variables

Required in `.env`:
- `DB_URL` — PostgreSQL connection string
- `JWT_SECRET` — secret for signing access tokens
- `ACCESS_TOKEN_TTL` — e.g. `1h` (default `1h`)
- `REFRESH_TOKEN_TTL` — e.g. `30d` (default `30d`)
- `VERIFICATION_TTL` — e.g. `1d` (default `1d`)

Cloudinary (required for photo uploads):
- `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET`

Optional weather overrides (defaults to Open-Meteo public API, Kenya):
- `WEATHER_GEOCODING_URL`, `WEATHER_FORECAST_URL`, `WEATHER_TIMEOUT_MS`
- `WEATHER_COUNTRY_CODE`, `WEATHER_TIMEZONE`
- `WEATHER_FORECAST_DAYS` (default `7`), `WEATHER_MAX_FORECAST_DAYS` (default `14`)

## Architecture

### Entry points
- `server.js` — starts the HTTP server
- `src/App.js` — configures Express: request logger → body parser → CORS → routes

### Database schema (PostgreSQL)
Key tables and their relationships:
- `users` — auth credentials, `username`, `email`, `phone`, `has_completed_setup`
- `farmer_profiles` — one per user; holds `county`, `subcounty`, `farming_type`, `interested_in_irrigation`
- `farms` — one per farmer profile (enforced by unique constraint on `farmer_profile_id`); holds water access fields and `total_size`
- `crops` — many per farm; unique on `(farm_id, crop_type)`
- `refresh_tokens` — hashed refresh tokens with expiry/revocation
- `verification_tokens` — hashed tokens for email/phone verification (type: `'email'` | `'phone'`)
- `crop_intelligence_history` — append-only log of every crop intelligence run; grouped by `run_group_id` (UUID)

All DB access goes through the single pool exported from `src/config/db.js`.

### Auth flow
- Access tokens are short-lived JWTs (Bearer). The `protect` middleware in `src/middleware/auth.middleware.js` verifies the JWT and attaches `req.user` (id, email, username) from a live DB lookup.
- Refresh tokens are random hex strings stored as SHA-256 hashes. Rotation happens on every `/auth/refresh` call (old token updated in place).
- Email and phone verification use separate token records in `verification_tokens`; changing the email/phone requires going through that flow.

### Route map
| Prefix | File | Notes |
|---|---|---|
| `/auth` | `auth.routes.js` | register, login, refresh, logout, verify email/phone, change password |
| `/users` | `user.routes.js` | `GET /me`, `PATCH /me`, `POST /onboarding/complete` |
| `/farms` | `farm.routes.js` | `PATCH /:farmId` — update farm name; ownership enforced via JOIN to `farmer_profiles` |
| `/crops` | `cropRoutes.js` | crop intelligence endpoints (see below) |
| `/dashboard` | `dashboard.routes.js` | aggregated farmer + farms + crops view |
| `/weather` | `weather.routes.js` | `GET /me` — weather for the user's saved location |

### Crop intelligence pipeline
The intelligence system lives entirely in `src/intelligence/` and is pure JS with no DB calls. It is invoked from `src/services/cropService.js`.

```
getCropIntelligence (controller)
  └─ generateCropInsights (cropService)
       └─ getCropInsights (intelligence/index.js)  ← switch on crop type
            └─ getMaizeInsights = createCropEngine({...})  ← shared engine template
```

`createCropEngine` (`src/intelligence/shared/cropEngineTemplate.js`) is the universal factory that all crop types use. It takes a set of crop-specific functions and returns a single insights function. To add a new crop, implement the same interface as `src/intelligence/crops/maize/` and register it in `intelligence/index.js`.

Each intelligence run is persisted to `crop_intelligence_history` (fire-and-forget — errors are caught and logged but do not fail the response).

**Crop stage resolution** (`src/intelligence/cropStage.js`): explicit `growthStage` string takes priority; otherwise derived from `plantingDate` using fixed day boundaries (≤7 Germination, ≤35 Vegetative, ≤60 Reproductive, else Maturity).

**Weather normalization** (`src/services/weather.service.js`): `normalizeWeatherData` converts the raw Open-Meteo response into the flat schema the intelligence engine expects. It summarises the first 3 forecast days as `forecastSummary`.

### Onboarding
`POST /users/onboarding/complete` is the single call that creates both `farmer_profiles` and `farms` (upsert on conflict). It also inserts initial `crops` rows if provided. After completion it sets `users.has_completed_setup = TRUE`.

### Module system
The project uses ESM (`"type": "module"` in package.json). All imports must use `.js` extensions.

---
> Source: [Leeobenga/mkulimaApp-backend](https://github.com/Leeobenga/mkulimaApp-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
