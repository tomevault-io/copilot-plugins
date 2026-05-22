## bazardhor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev              # Next.js dev server (Turbopack)
npm run build            # Production build (Turbopack)
npm run start            # Run built app
npm run lint             # ESLint (next/core-web-vitals + next/typescript)
npm run validate:env     # node scripts/validate-env.js
npm run setup:env        # Copy .env.example → .env.local
npm run check:components # Static check: scan src/**/*.{tsx,jsx} for missing/unnecessary "use client"
```

There is no test runner configured (the README mentions `npm run test`, but no scripts/tooling exist).

`next.config.ts` sets `eslint.ignoreDuringBuilds: true` and `typescript.ignoreBuildErrors: true` — `npm run build` will succeed despite lint/type errors. Run `npm run lint` and `npx tsc --noEmit` explicitly when you need a real check.

The default API base URL (when `NEXT_PUBLIC_API_URL` is unset) is `https://bazardor.mainul.tech/api`, defined in both `src/lib/api/client.ts` and `src/lib/api/fetch-client.ts`. `next.config.ts` whitelists `bazardor.mainul.tech` and `images.unsplash.com` for `next/image`.

## Architecture

Next.js 15 App Router PWA. Path alias `@/*` → `./src/*`. UI follows shadcn-ui conventions (`components.json`: style `new-york`, base color `neutral`, Lucide icons) — `src/components/ui` holds the primitives.

### Zone gating (load-bearing — read before touching API code)

Almost every backend endpoint requires a `zoneId` request header. This is enforced in **two** places:

- [src/lib/api/client.ts](src/lib/api/client.ts) — axios request interceptor rejects with an `AxiosError` of code `ZONE_REQUIRED` if no `zoneId` cookie/localStorage value is present.
- [src/lib/api/fetch-client.ts](src/lib/api/fetch-client.ts) — server-side `fetch` wrapper throws / returns `null` when the zoneId header is missing.

Only paths in `ZONE_OPTIONAL_ENDPOINTS` (currently just `/config/get-zone`) are exempt. Any new "open" endpoint must be added to **both** sets.

[src/providers/zone-provider.tsx](src/providers/zone-provider.tsx) bootstraps the zone on mount: it tries `localStorage.user_lat/user_lng` first, then IP geolocation (`ipapi.co`, falling back to `ipwho.is`), then a hardcoded Dhaka default (`23.8103, 90.4125`). The resulting zone is persisted into both `Cookies.zoneId` (7-day expiry, used server-side) and `localStorage.user_zone`. `usePreciseLocation()` re-runs the lookup with `navigator.geolocation`.

### Two API clients (client vs. server)

- **[src/lib/api/client.ts](src/lib/api/client.ts)** — axios instance for client components. Auto-attaches `Authorization: Bearer <auth_token>` from `localStorage` and `zoneId` from the cookie. Used by everything in [src/lib/api/services/](src/lib/api/services/) and the React Query hooks in [src/lib/api/hooks/](src/lib/api/hooks/).
- **[src/lib/api/fetch-client.ts](src/lib/api/fetch-client.ts)** — native `fetch` wrapper for Server Components / RSC. Supports `next: { revalidate }` for ISR. The caller must pass `headers: { zoneId }` explicitly. Used by [src/lib/api/services/server/](src/lib/api/services/server/).

[src/lib/api/endpoints.ts](src/lib/api/endpoints.ts) is a constants file but it has drifted from what individual service files actually call (e.g. `categoriesApi.getAll` hits `/categories/list`, not `/categories`). When wiring new requests, trust the service file and the network tab over the constants file.

#### Generic response wrappers

Three response shapes are defined in [src/lib/api/types.ts](src/lib/api/types.ts) — pick the right one when typing a new call:

- `ApiResponse<T>` — `{ data: T, message?, success }`. Used for most internal `/api/*` Next.js route handlers and our own service layer (e.g. `apiClient.post<ApiResponse<AuthResponse>>('/auth/login', ...)` then read `response.data.data`).
- `BackendApiResponse<T>` — `{ response_code, message, total_size, limit, offset, data: T, errors }`. The shape returned by the upstream `bazardor.mainul.tech` API for list endpoints.
- `PaginatedResponse<T>` — `{ data: T[], pagination: { page, limit, total, totalPages } }`. Used by paginated client services.

Important: `fetchClient<T>` (the server-side fetch wrapper) **auto-unwraps `data`** when the response is shaped like `{ data, ... }` and `success !== false`. So with `fetchClient`, type the generic as the **inner** payload (`fetchClient<Category[]>`), not the wrapper. With `apiClient` (axios) there is no auto-unwrap — type the full wrapper and read `response.data.data` yourself.

When writing a new route handler under [src/app/api/](src/app/api/), return `NextResponse.json<ApiResponse<T>>({ success, data, message }, { status })` so callers can rely on the same shape.

#### Convention: explain every generic you introduce

Whenever you add a generic type parameter (e.g. `<T>` on a function, hook, component, or new wrapper interface), leave a one-line comment directly above the declaration that states:

1. **Why** the generic is needed — what concrete reuse / type-flow problem it solves that a fixed type wouldn't.
2. **How** callers are expected to instantiate it — what shape `T` should be, and any constraints (e.g. "`T` must be the unwrapped payload because `fetchClient` strips the outer `{ data }`").

Example:
```ts
// Why: lets services share one wrapper type instead of redeclaring `{ data, success, message }` per endpoint.
// How: T is the payload AFTER unwrapping — for paginated endpoints use PaginatedResponse<Item> as T.
export interface ApiResponse<T> { data: T; message?: string; success: boolean }
```

This is one of the few places in this repo where comments are required by default. Reusing an existing generic (`ApiResponse<Foo>`) does **not** need a comment — only declaring a new generic parameter does.

### State boundary

Two parallel state systems with non-overlapping responsibilities — don't merge them:

- **Zustand** ([src/store/app-store.ts](src/store/app-store.ts)) — auth (user, isAuthenticated), bootstrapped server config (`appConfig`, `settings`, `generalConfig`), favorites, search/filter UI state, location, modals, toasts. Wrapped in `devtools + persist + immer`. Persist key: `fresh-market-finder-store`. Selector hooks in [src/store/hooks.ts](src/store/hooks.ts) (`useAuth`, `useSearch`, `useFavorites`, `useUI`, `useLocation`, `useNotifications`).
- **React Query** ([src/providers/query-provider.tsx](src/providers/query-provider.tsx)) — all server-cache data (markets, items, categories, banners, reviews). Defaults: `staleTime` 1 min, no retry on 4xx, mutations don't retry.

`hasHydrated` flag on the store gates effects that read persisted state. [src/components/auth/auth-context.tsx](src/components/auth/auth-context.tsx) waits for both `hasHydrated` AND `zone?.id` before calling `/auth/me` — touch this ordering carefully or you'll race the zone guard.

Auth registration is split intentionally (see [docs/auth-zustand-changes.md](docs/auth-zustand-changes.md)): `useRegister()` only updates the React Query cache; the modal calls `useAppStore.getState().login(...)` itself.

### Server-side config bootstrap

[src/app/layout.tsx](src/app/layout.tsx) is `async`. It reads the `zoneId` cookie via `next/headers`, calls `configServerApi` for `/config/app`, `/config/settings`, `/config` in parallel (revalidated hourly), and hands the results to [src/providers/config-bootstrap.tsx](src/providers/config-bootstrap.tsx) — a small `"use client"` component that seeds the Zustand store on mount. If there's no zoneId cookie yet (first visit), all three are `null` and the client-side ZoneProvider catches up after detection.

### Provider order

In [src/app/layout.tsx](src/app/layout.tsx):

```
QueryProvider
└─ ConfigBootstrap (seeds Zustand)
   └─ ZoneProvider          (provides zone, gates dependent calls)
      └─ SearchProvider
         └─ AuthProvider     (waits for zone + hydration)
            └─ AddItemProvider
               └─ {children}
```

This order matters: AuthProvider depends on ZoneProvider, ZoneProvider depends on QueryProvider (uses `useGetZoneMutation`).

### App Router conventions

- Routes live under [src/app/](src/app/) (`markets/`, `markets/[id]/`, `markets/compare/`, `category/`, `category/[slug]/`, `items/`, `profile/`, `about/`).
- Per-route private components go in `_components/` directories (e.g. [src/app/_components/](src/app/_components/) for the homepage). Cross-route shared components live in [src/components/](src/components/).
- When adding a component, run `npm run check:components` — it flags files that use hooks/event handlers/browser APIs/swiper/framer-motion/`createContext` without `"use client"`, and conversely flags `"use client"` files that look like they could be Server Components.

### PWA

`next-pwa` is wired in [next.config.ts](next.config.ts) (disabled in development). Service worker output lands in `public/`. Runtime caching rules cover Unsplash, local images, and Google Fonts. The manifest is at [public/manifest.json](public/manifest.json) and is referenced from `metadata.manifest` in the root layout.

---
> Source: [ziaul888/bazarDhor](https://github.com/ziaul888/bazarDhor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
