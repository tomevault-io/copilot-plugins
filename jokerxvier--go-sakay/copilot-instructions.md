## go-sakay

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Go Sakay — an Uber-inspired ride-hailing driver app built with Expo (SDK 55), React Navigation v7, and TypeScript strict mode. Dark-mode-first design with NativeWind v4 styling. Package manager is Yarn 4.

## Development commands

```bash
npx expo start              # Start Expo dev server
npx expo start --ios        # iOS simulator
npx expo start --android    # Android emulator
npx tsc --noEmit            # Type-check (no lint/test/format scripts yet)
```

## Architecture

### Routing — Expo Router (file-based)

```
app/_layout.tsx              # Root Stack — ThemeProvider, global.css, QueryClientProvider, font loading
app/(auth)/_layout.tsx       # Auth group — headerless, dark bg
app/(auth)/login.tsx         # OTP login
app/(auth)/register.tsx      # Multi-step registration
app/(auth)/onboarding.tsx    # Onboarding slides
app/(tabs)/_layout.tsx       # Bottom tab navigator (Home, Notifications, Ratings, Account)
app/(tabs)/index.tsx         # Home — main dashboard with trip state machine
app/(tabs)/earnings/         # Nested stack: dashboard, trip-history, cash-out, payout, trip/[id]
app/(tabs)/account/          # Nested stack: profile, documents, preferences, support, vehicle-details
app/modal.tsx                # Modal screen
```

Root Stack has three screens: `(tabs)`, `(auth)`, and `modal`. Initial route is `(tabs)`. Typed routes are enabled via `app.config.ts`.

### State management — Zustand stores (`stores/`)

Seven stores drive the app state:

- **`auth-store`** — user, tokens, auth persistence via `expo-secure-store`
- **`driver-store`** — location, driver mode (online/offline), daily stats
- **`trip-store`** — full trip lifecycle state machine (incoming request → active trip → rating)
- **`navigation-store`** — turn-by-turn nav state (route, current step, maneuver)
- **`earnings-store`** — earnings data, trip history, payout info
- **`preferences-store`** — user prefs (sound, haptics, notifications)
- **`notification-store`** — notifications list

Pattern: `create<State>((set, get) => ({ ...properties, ...actions }))`. Use selectors in components: `useDriverStore((s) => s.currentLocation)`.

### Trip lifecycle (core domain)

`TripStatus` in `shared/types/trip.ts` defines the state machine: `idle → incoming → en_route → arrived → in_progress → completed`. The `trip-store` manages transitions. `DriverMode` in `shared/types/driver.ts`: `offline | online`.

### API layer — `lib/api/`

Mock-first architecture. `client.ts` exports `mockDelay()`, `mockSuccess<T>()`, `mockError()` for dev. Real endpoints exist for directions (Google Directions API). Module files: `auth`, `driver`, `trip`, `directions`, `earnings`, `ratings`, `notifications`, `documents`, `support`, `vehicle`.

### Custom hooks (`hooks/`)

24 hooks organized by concern: data fetching, location tracking (5s GPS interval), trip state (ride requests, navigation, fare meter), voice guidance, and dev simulation.

### Key modules

- `lib/utils.ts` — `cn()` merges classes via clsx + tailwind-merge. Use for all conditional/composed className props.
- `lib/icons.ts` — Centralized lucide-react-native re-exports (~75 icons) with `DEFAULT_STROKE_WIDTH` (1.5) and `DEFAULT_ICON_SIZE` (24). Always add new icons here; never import directly from lucide-react-native.
- `lib/validation.ts` — Validators: phone, email, age (≥18), plate number, OTP.
- `lib/map-config.ts` — Map configuration and API keys.
- `lib/voice-instructions.ts` — Navigation voice guidance text.
- `components/ui/` — Primitives: Button, Text, Input, Label, Checkbox, DatePickerInput. Check here before building custom.
- `components/dev/DevTripToolbar.tsx` — In-app simulation controls (guarded by `__DEV__`).

### Shared types and utils (`shared/`)

`shared/types/` has domain types: `api.ts` (ApiResponse<T>), `auth.ts`, `driver.ts` (DriverMode, LatLng), `trip.ts` (TripStatus, RideRequest, FareBreakdown), `earnings.ts`, `ratings.ts`, `notifications.ts`, `payout.ts`, `navigation.ts`. `shared/utils/` has `geo.ts` (haversine) and `polyline.ts` (Google Maps encoding).

### Path alias

`@/*` maps to project root via `tsconfig.json`. Example: `import { cn } from '@/lib/utils'`.

**Important:** `CODING_STANDARDS.md` examples use `~/` but the actual tsconfig uses `@/`. Always use `@/`.

### NativeWind v4

- Config: `tailwind.config.js` with `nativewind/preset`. All semantic color tokens, custom `fontSize` (`2xs`), `borderRadius` (`xs`), `boxShadow` (`accent`, `success`) defined there.
- CSS: `global.css` imported in `app/_layout.tsx`.
- Content paths: `./app/**`, `./components/**`, `./lib/**`.

## Agent team

Read `AGENTS.md` before cross-cutting tasks. It defines four agents (Orchestrator, TypeScript Expert, RN Architect, Backend API Agent) with lane boundaries and tool permissions. Key rule: TypeScript Expert runs first (types before implementation), then RN Architect and Backend API Agent in parallel.

## Critical conventions

Read `CODING_STANDARDS.md` for the full ruleset. The most commonly violated rules:

- **`className` only** — `StyleSheet.create` is banned. Use NativeWind semantic tokens, never hardcode hex.
- **Class order**: layout → sizing → spacing → visual → typography → state variants.
- **Icons**: Import from `@/lib/icons`, not directly from lucide-react-native. No `@expo/vector-icons`.
- **No `any`** — strict mode, use `unknown` + narrowing. No `@ts-ignore`.
- **Components**: accept + forward `className` prop. No business logic — delegate to services/hooks.
- **Files**: `kebab-case.ts` for non-components, `PascalCase.tsx` for components.
- **Exports**: All need JSDoc.
- **State**: Server state → TanStack Query. Global client state → Zustand. No Redux, no Context for server state.
- **Lists**: FlashList over FlatList for 20+ items. Images via expo-image.
- **Touch targets**: minimum `w-11 h-11` (44×44px).
- **Import order**: Node → external → internal aliases (`@/`) → relative → type imports.

## Theme tokens

Source of truth: `tailwind.config.js`. Key groups:

- **Surfaces**: `background` (DEFAULT/elevated/overlay/subtle)
- **Text**: `foreground` (DEFAULT/secondary/tertiary/inverse)
- **CTA**: `primary` (DEFAULT/foreground/muted) — white-on-black buttons
- **Accent**: `accent` (DEFAULT/foreground/muted/strong) — surge amber (#F0B429)
- **Status**: `success`, `destructive`, `info` (each with muted variant)
- **Domain**: `driver` (DEFAULT/offline/on_trip), `map` (pickup/dropoff/route/eta)
- **Borders**: `border` (DEFAULT/strong/subtle)

Full token-to-hex mapping and className patterns are in `AGENTS.md` Theme section.

## Git

- **Never** add `Co-authored-by: Claude` or any AI attribution trailer to commits
- Conventional Commits: `feat(scope): description`
- Branch naming: `feat/`, `fix/`, `chore/`, `refactor/`

## Supplementary rules

- `.agents/skills/vercel-react-best-practices/` — React performance patterns
- `.agents/skills/vercel-react-native-skills/` — React Native optimizations (animations, lists, navigation)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jokerxvier) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
