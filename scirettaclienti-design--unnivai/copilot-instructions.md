## unnivai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # Start dev server on port 5173 (strict port)
npm run build     # Production build
npm run lint      # ESLint check
npm run preview   # Preview production build
```

```bash
npm test                  # watch mode (Vitest)
npm run test:run          # single pass (CI)
npm run test:ui           # browser UI at localhost
npm run test:coverage     # coverage report (v8)
```

**Stack:** Vitest 3 + jsdom + @testing-library/react + @testing-library/jest-dom
**Location:** `src/__tests__/` mirrors `src/` structure.
**Mocks:** `src/test/mocks/supabase.js` exports `createQueryBuilder()`, `createSessionMock()`, `NO_SESSION`. Use `vi.mocked(supabase.from).mockReturnValue(createQueryBuilder({data, error}))` per test.
**Setup:** `src/test/setup.js` stubs `navigator.geolocation` and silences `console.error/warn` from dataService migration warnings.
**fetch mocking:** Use `vi.stubGlobal('fetch', vi.fn().mockResolvedValue({ok, json}))` for Nominatim calls in userContextService tests. Always pair with `vi.unstubAllGlobals()` in `afterEach`. Use `vi.resetAllMocks()` (not `clearAllMocks`) in `beforeEach` so mock implementations don't bleed between tests.

## Environment Variables

Copy `.env.example` to `.env` and fill in:
```
VITE_SUPABASE_URL=
VITE_SUPABASE_ANON_KEY=
VITE_MAPBOX_TOKEN=
VITE_OPENAI_API_KEY=   # Optional: enables real AI itinerary generation
```

All env vars must be prefixed with `VITE_` to be accessible via `import.meta.env`.

## Architecture Overview

**DoveVai** is a mobile-first React 19 + Vite SPA for Italian tourism. It connects travellers (esploratori), local guides, and businesses. The UI language is Italian.

### Provider Stack (outermost → innermost)
```
QueryClientProvider (React Query, 5-min stale cache)
  └── AuthProvider (Supabase auth + role)
        └── CityProvider (active city, GPS vs manual)
              └── Router → ErrorBoundary → Suspense → Routes
```

### Role-Based Routing

`RootDispatcher` (at `/`) redirects authenticated users by role:
- `explorer` / `user` → `/dashboard-user`
- `guide` → `/dashboard-guide`
- `business` → `/dashboard-business`
- unauthenticated → `/` (Landing page)

`RoleGuard` wraps protected route groups and enforces role access. Role is read from `user.user_metadata.role`, with fallback to `localStorage.getItem('unnivai_role')`.

All pages are lazy-loaded via `React.lazy()`.

### Path Alias

`@` resolves to `./src/` (configured in `vite.config.js`).

### Key Contexts

| Context | File | What it provides |
|---|---|---|
| `AuthContext` | `src/context/AuthContext.jsx` | `user`, `role`, `isAuthenticated`, `signOut`, `resetPassword`, `isPasswordRecovery` |
| `CityContext` | `src/context/CityContext.jsx` | `city` (default `'Roma'`), `setCity` (marks as manual), `isManual`, `resetToGPS` |

### Key Services (singleton class instances)

| Service | File | Responsibility |
|---|---|---|
| `dataService` | `src/services/dataService.js` | Supabase CRUD: tours, bookings, favorites, notifications, activities, businesses. Exposes `mapTourToUI()` which normalizes DB rows to the UI contract. |
| `aiRecommendationService` | `src/services/aiRecommendationService.js` | AI itinerary generation. Uses OpenAI GPT (if `VITE_OPENAI_API_KEY` set) or falls back to local mock data. Also handles business analysis via vision model. |
| `userContextService` | `src/services/userContextService.js` | Resolves effective city from manual override → GPS → Supabase profile → localStorage. Does reverse geocoding via Nominatim (OSM). |
| `weatherService` | `src/services/weatherService.js` | Provides weather data for the current city. |

### Key Hooks

- `useUserContext()` — composes GPS, auth, and CityContext into a single user context object (`city`, `firstName`, `temperatureC`, `toursCount`, `isGuest`). Used by dashboards and TopBar.
- `useEnhancedGeolocation()` — wraps browser GPS API, reverse geocodes to city name via Nominatim, falls back to Roma on error/denial.
- `useUserNotifications()` — `src/hooks/useUserNotifications.js`

### Supabase Schema

All migrations live in `supabase/migrations/`. Apply them in filename order. RLS base policies in `supabase/rls_policies.sql`.

**Canonical tables and their authoritative columns:**

| Table | PK | Key columns | Notes |
|---|---|---|---|
| `tours` | `id` | `city`, `guide_id`, `is_live`, `price_eur`, `duration_minutes`, `image_urls` | `price_eur` added by `migration_add_filters.sql` |
| `profiles` | `id` (= auth user id) | `full_name`, `avatar_url`, `bio`, `current_city`, `username` | Synced with `auth.users` |
| `bookings` | `id` | `tour_id`, `user_id`, `status` (`pending_request`→`accepted`/`declined`) | |
| `guide_requests` | `id` | `user_id`, `guide_id`, `tour_id`, `status` (`open`/`accepted`/`declined`/`completed`), `created_at`, `city`, `request_text`, `user_name`, `duration`, `category`, `notes` | Columns added across two migrations |
| `guides_profile` | `id` | `user_id` (unique), `license_number`, `piva`, `bio`, `status`, `type` (`pro`/`host`), `operating_cities[]` | Created in `20260303` migration |
| `explorers` | `id` (= auth user id) | `tours_completed`, `km_walked` | Created in `20260303` migration |
| `user_photos` | `id` | `user_id`, `tour_id`, `media_url` | Created in `20260303` migration |
| `favorites` | `id` | `user_id`, `tour_id` | Unique constraint on `(user_id, tour_id)` |
| `notifications` | `id` | `user_id`, `is_read` (canonical), `read_at`, `action_url`, `action_data`, `city_scope` | |
| `activities` | `id` | `city`, `name`, `latitude`, `longitude`, `category`, `tier`, `owner_id` | |
| `businesses_profile` | `id` | `company_name`, `city`, `category_tags[]`, `ai_metadata` (JSONB), `location` (PostGIS POINT) | |

### Runtime Schema Validation

`src/lib/schemas.js` exports three Zod v4 schemas that act as the authoritative data contracts:

| Schema | Validated in | Purpose |
|---|---|---|
| `TourUISchema` | `dataService.mapTourToUI()` — every call site | Catches DB→UI shape breaks (NaN rating, null images, missing guide fields) |
| `BookingInputSchema` | `dataService.createBooking()` before DB write | Catches bad form data (string price, missing tourId, invalid date format) before the silent-success fallback hides it |
| `NotificationUISchema` | `dataService.getNotifications()` per item | Enforces unified shape; surfaces the realtime vs REST divergence as logged errors |

Validation is **non-blocking** via `validateData(schema, data, label)` (defined in `dataService.js`). A failed parse logs `🚨 [Schema] Validation failed [label]: {fieldErrors}` to the console but always returns the original data so the UI is never broken by a schema change.

### Known Column Contract Issues (do not re-introduce)

- **`avatar_url` vs `avatar_emoji`**: RESOLVED. `mapTourToUI` now checks `avatar_url` first, then `avatar_emoji`, then `'👋'`.
- **`is_read` is the canonical field** for notification read state. Do not use `read` or `read_at` as primary boolean. `subscribeToNotifications()` in `dataService.js` still uses `n.is_read` — keep it that way.
- **`price_eur` is prioritized** over `price` in `mapTourToUI()`. Both columns exist; `price_eur` requires `migration_add_filters.sql` to be applied.
- **`getPendingBookingsForGuide`**: RESOLVED (ALTO-3). Now JOINs `profiles:user_id(full_name, avatar_url)` — `userName` and `userAvatar` are available in the result without a follow-up query.
- **Explore.jsx direct supabase query**: RESOLVED (CRITICO-1). Now uses `profiles:guide_id(...)` JOIN and routes DB rows through `dataService.mapTourToUI()`. JSX uses `experience.imageUrl` (not `.image`).

### AI Itinerary Flow

`aiRecommendationService.generateItinerary()`:
1. Fetches simulated/real weather
2. If `VITE_OPENAI_API_KEY` is set → calls `gpt-3.5-turbo` with partner business context injected into the system prompt
3. Validates/geocodes AI-returned coordinates via Mapbox if missing
4. Falls back to `generateItineraryLocal()` with hardcoded city POIs + partner businesses from Supabase

Business injection into itineraries: `dataService.getBusinessesByCityAndTags()` scores `businesses_profile` rows using tag affinity mapping and injects up to 3 sponsors per tour. Missing coordinates are resolved via Nominatim and cached back to Supabase.

### City Data Flow

The `city` value is resolved through a priority chain managed by `userContextService`:
1. **Manual** (user selects in TopBar → `CityContext.setCity()`)
2. **GPS** (`useEnhancedGeolocation` → Nominatim reverse geocode)
3. **Supabase profile** (`profiles.current_city` for authenticated users)
4. **localStorage** (`user_city` key)
5. **Fallback** hardcoded `'Roma'`

`useUserContext()` merges the GPS hook output with CityContext and sanitizes coordinate strings that can leak from GPS failures (pattern `"Lat: X.XX, Lon: Y.YY"`).

`CityContext.resetToGPS()` exists but is not wired to any UI button.

### UI Conventions

- Styling: Tailwind CSS v3 with `tailwindcss-animate` plugin. `clsx` + `tailwind-merge` for conditional classes (via `src/lib/utils.js`).
- Animations: Framer Motion.
- Icons: Lucide React.
- Map: `react-map-gl` wrapping `mapbox-gl`, token via `VITE_MAPBOX_TOKEN`.
- Brand color: orange-500 (`#f97316`). Loading spinners use `border-orange-500`.
- The app displays Italian text throughout. City defaults to `'Roma'` as fallback everywhere.
- Demo/fallback data lives in `src/data/demoData.js`.

---
> Source: [scirettaclienti-design/unnivai](https://github.com/scirettaclienti-design/unnivai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
