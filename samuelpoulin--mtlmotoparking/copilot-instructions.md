## mtlmotoparking

> Brief repo orientation for automated agents. Keep this file short; use tools for details.

# AGENTS

Brief repo orientation for automated agents. Keep this file short; use tools for details.

## What this app is
- Next.js App Router app to find Montreal motorcycle parking spots.
- Core UX: map + marker drawer, search by address, community spot updates (photo/fullness), favourites, auth-gated settings.

## Stack (current)
- Next.js 16, React 19, TypeScript (strict), Tailwind v4.
- Drizzle ORM + Postgres (Neon).
- better-auth (Google/Facebook OAuth, admin + testUtils plugins).
- Maps: maplibre/react-map-gl + Mapbox search/geocoding.
- Client state/data: Zustand + TanStack Query.
- Uploads: Cloudinary (signed uploads + ownership verification).
- i18n: next-intl (`en`, `fr`), analytics: PostHog.

## Run
- `bun run dev` / `bun run build` / `bun run start`
- `bun run lint`
- `bun run test` (Playwright E2E)

## High-value paths
- Pages: `app/[locale]/page.tsx`, `app/[locale]/map/page.tsx`, `app/[locale]/settings/page.tsx`, `app/[locale]/sign-in/page.tsx`
- API: `app/api/**` (auth, contributions, favourites, user, Cloudinary signature)
- Data pipeline: `src/lib/api/parkings.ts`, `src/lib/api/locations.ts`
- DB schema: `src/lib/db/schema.ts`
- Auth: `src/lib/auth.ts`, `src/lib/auth-client.ts`
- Map UI: `src/components/ParkingMap.tsx`, `src/components/ParkingSpotDrawer/*`
- i18n: `src/i18n/*`, messages in `messages/en.json` + `messages/fr.json`

## Behavior and constraints
- Map page server pipeline is cached for 12h (`unstable_cache` in `app/[locale]/map/page.tsx`).
- Final displayed dataset uses latest row per location and `rep_description = "Réel"`.
- Addresses are reverse-geocoded via Mapbox when missing/stale (48h staleness window).
- Contributions: max 1 per user/parking per 18h, latest 5 shown, admin bypasses limit.
- Favourites: max 5 per user.

## Tests and CI
- Playwright suite exists under `tests/` with admin/user/guest projects.
- Config: `playwright.config.ts`; CI workflow: `.github/workflows/playwright.yml`.

## Agent notes
- Prefer reading code over assumptions; this file is intentionally non-exhaustive.
- Do not edit real `.env`; document env changes in `.env.template`.
- Use `bun x drizzle-kit generate` for migrations; do not run/push DB migrations unless user asks.

---
> Source: [SamuelPoulin/mtlmotoparking](https://github.com/SamuelPoulin/mtlmotoparking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
