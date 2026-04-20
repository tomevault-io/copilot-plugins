## food-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
yarn start           # Start Expo dev server
yarn start-clear     # Start with cleared cache
yarn android         # Run on Android
yarn ios             # Run on iOS
yarn web             # Run on web
yarn lint            # ESLint + Prettier checks
yarn format          # Auto-fix with ESLint + Prettier
yarn prebuild        # Expo prebuild (native folders)
```

No test runner is configured in this project.

## Architecture

### Routing (Expo Router v6 — file-based)

`app/` maps directly to routes. Authentication state (from Zustand) drives redirects in `app/_layout.tsx`:

- **No token** → `app/(auth)/` — intro slider, sign-in, sign-up, password recovery, phone/identity verification
- **Has token** → `app/(tabs)/` — Home, Search, Order, Profile
- **Extra** → `app/(extra)/` — supplemental screens (e.g. find-location)

Route constants are typed in `constants/route.ts` (`ROUTES.AUTH.*`, `ROUTES.TABS.*`, `ROUTES.EXTRA.*`).

### State Management

Three Zustand stores + TanStack Query:

1. **`stores/auth.ts`** — auth session. `boot()` runs a silent refresh on startup; `signIn/signOut` manage the session; `refreshAccessToken()` is called by the Axios interceptor directly (not via React).
2. **`stores/app/store.ts`** — app-wide state (user location). Uses `zustand/middleware` `persist` backed by AsyncStorage. Only the location field is persisted.
3. **`stores/bottom-sheet/store.ts`** — controls which bottom sheet is open globally. `contentType` is one of `filter_product | search_product | search_location | list_best_partner`; `meta` carries data between components. The `AppBottomSheet` organism reads this store and renders the right sheet content.
4. **TanStack Query** (`libs/api/tanstack-query.ts`) — server/cached data. 45s stale time, 1 retry, refetches on AppState focus change and NetInfo reconnect.

### API Layer (`libs/api.ts`)

Axios instance pointing at `https://api-ec.artstack.online`. Interceptors:
- **Request** — injects `Authorization: Bearer <accessToken>` from the auth store.
- **Response** — on 401, acquires a refresh lock (prevents concurrent refresh races), calls `refreshAccessToken()`, then retries the original request once. Failed refresh clears auth state.

Refresh token is persisted via `expo-secure-store` (`libs/secure.ts`: `saveRefresh`, `loadRefresh`, `deleteRefresh` with `WHEN_UNLOCKED` access level).

### Styling

NativeWind v5 + Tailwind v4. Global CSS lives in `app/globals.css` (defines the `@theme` color palette as CSS custom properties). Use Tailwind utility classes directly on components. Prettier auto-sorts class names via the Tailwind plugin.

**Dual color system** — colors are defined in two parallel places that must stay in sync:
- `constants/Colors.ts` — TypeScript object (`COLOR.primary[500]`, etc.) with `getColor<K>()` helper; used in runtime JS (e.g. passing colors as props).
- `app/globals.css` — same palette as Tailwind CSS vars (`bg-primary-500`, `text-neutral-100`); used via NativeWind class names.

Path alias `@/` resolves to the repo root (configured in `tsconfig.json` and `babel.config.js`).

Class-name merging utility: `utils/style.ts` exports `cn()` (wraps `clsx` + `twMerge`).

### Component Structure

```
components/
  atoms/       # Primitive UI (Button, AppImage, Input, Icons, Slider, Loading)
  molecules/   # Composite UI (AppTabBar, UITabs, Combobox, ConfirmModal, form helpers)
  orangism/    # Container-level patterns (AppBottomSheet, AppTopSheet)
features/      # Feature-specific components, co-located with feature logic
  tabs/home/components/   # Home tab sections (category, filter, partner, location, search)
  location/               # LocationCombobox
```

`components/orangism/AppBottomSheet` is the global bottom sheet manager — it reads the bottom-sheet store and routes to the appropriate sheet content component. It handles keyboard visibility, Android back navigation, and gesture dismissal.

### Services & Queries

```
services/      # Raw API call functions (food/, location/)
queries/       # TanStack Query hooks that wrap services (e.g. queries/location.ts → useSearchLocation)
libs/schema/   # Zod schemas for form validation (auth.ts, sign-in.ts, sign-up.ts)
libs/seed/     # Mock data (cities, food-category, partner, product) — used while API is not wired up
```

Form validation pattern: define a Zod schema in `libs/schema/`, infer the type with `z.infer<>`, pass `zodResolver(schema)` to `useForm`.

### Confirmation Dialog

`providers/ConfirmProvider.tsx` exposes a promise-based confirm API via context. Wrap the tree with `<ConfirmProvider>` and use `useConfirm().confirm(options)` — resolves `true/false` when the user responds. The modal renders centrally in the provider.

### Key Libraries

| Purpose | Library |
|---|---|
| Lists | `@shopify/flash-list` |
| Bottom sheets | `@gorhom/bottom-sheet` |
| Icons | `lucide-react-native` |
| Forms | `react-hook-form` + `zod` |
| Animations | `react-native-reanimated` v4 |
| Gestures | `react-native-gesture-handler` |
| OTP input | `input-otp-native` |
| Location search | Photon API (`services/location/photon.ts`) |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/minhtrung0110) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
