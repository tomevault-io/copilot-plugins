## menuz-mobile

> - Expo app with Expo Router v6 and TypeScript strict mode.


## Project overview

- Expo app with Expo Router v6 and TypeScript strict mode.
- React Native stack with web support via `react-native-web`.
- Path alias: `@/*` maps to repo root per `tsconfig.json`.

## Directory structure

- `app/` — file-based routing (screens):
  - Root layout: `app/_layout.tsx`.
  - Tabs group: `app/(tabs)/` with `app/(tabs)/_layout.tsx` and screens `index.tsx`, `favorites.tsx`, `search.tsx`, `profile.tsx`.
  - Dynamic routes: `app/restaurant/[id].tsx`, `app/category/[type].tsx`, `app/promotion/[id].tsx`, `app/restaurant-sharing/[id].tsx`.
  - Other screens: `auth.tsx`, `help-support.tsx`, `my-reviews.tsx`, `notification-preferences.tsx`, `profile.tsx`, `suggest-restaurant.tsx`, `+not-found.tsx`.
- `components/` — reusable UI and feature components:
  - `components/layouts/ScreenLayout.tsx` base wrapper.
  - `components/ui/*` primitives: `Badge`, `FilterButton`, `Header`, `LoadingSpinner`, `LocationPicker`, `MenuSection`, `PromoBanner`, `RatingBadge`, `RestaurantCard`, `ReviewCard`, `SearchBar`, `Skeleton`, `Toast`, `UserAvatar`, etc.
  - `components/onboarding/*` stepper/wrapper.
  - `components/sharing/*` (QR code generator/scanner, social sharing card).
  - `components/restaurant-sharing/[id].tsx` route-specific helper.
  - `components/sections/HeroSection.tsx` homepage hero.
- `hooks/` — cross-cutting hooks: `useFrameworkReady`, `useKeyboardNavigation`, `useToast`.
- `data/mockData.ts` — mock content for development and examples.
- `assets/images/*` — app imagery and icons.

## Conventions

- Screens import from `expo-router` (`Link`, `useRouter`, `useLocalSearchParams`).
- Layouts should compose `ScreenLayout` for consistent padding/status bar.
- Prefer `components/ui/*` for primitives. Extract new primitives if reused ≥2 screens.
- Use `useToast` for user feedback; prefer non-blocking UX.
- Keep dynamic route params typed via `useLocalSearchParams<{ id: string }>()` style.

## Patterns and utilities

- Lists: use `FlatList`/`SectionList` with stable keys; memoize renderItem when heavy.
- Loading/Empty states: reuse `Skeleton` and `EmptyState` components.
- Sharing/QR: follow patterns in `components/sharing/QRCodeGenerator.tsx` and `QRCodeScanner.tsx`; provide web fallbacks.
- Location/camera permissions: request at point of use with respective `expo-*` APIs.

## Web considerations

- Ensure components render on web; guard native-only APIs.
- Use `react-native-svg` and `react-native-webview` that are configured in `package.json`.

## Performance notes

- Memoize expensive subtrees; split components where needed.
- Avoid unnecessary re-renders in tab screens; keep state local to screens.

## Quality

- Run `expo lint` before committing; avoid suppressing TypeScript errors.
- Follow import order: node/external → `@/` alias → relative.

## Project overview

- Expo app with Expo Router v6 and TypeScript strict mode.
- React Native stack with web support via `react-native-web`.
- Path alias: `@/*` maps to repo root per `tsconfig.json`.

## Directory structure

- `app/` — file-based routing (screens):
  - Root layout: `app/_layout.tsx`.
  - Tabs group: `app/(tabs)/` with `app/(tabs)/_layout.tsx` and screens `index.tsx`, `favorites.tsx`, `search.tsx`, `profile.tsx`.
  - Dynamic routes: `app/restaurant/[id].tsx`, `app/category/[type].tsx`, `app/promotion/[id].tsx`, `app/restaurant-sharing/[id].tsx`.
  - Other screens: `auth.tsx`, `help-support.tsx`, `my-reviews.tsx`, `notification-preferences.tsx`, `profile.tsx`, `suggest-restaurant.tsx`, `+not-found.tsx`.
- `components/` — reusable UI and feature components:
  - `components/layouts/ScreenLayout.tsx` base wrapper.
  - `components/ui/*` primitives: `Badge`, `FilterButton`, `Header`, `LoadingSpinner`, `LocationPicker`, `MenuSection`, `PromoBanner`, `RatingBadge`, `RestaurantCard`, `ReviewCard`, `SearchBar`, `Skeleton`, `Toast`, `UserAvatar`, etc.
  - `components/onboarding/*` stepper/wrapper.
  - `components/sharing/*` (QR code generator/scanner, social sharing card).
  - `components/restaurant-sharing/[id].tsx` route-specific helper.
  - `components/sections/HeroSection.tsx` homepage hero.
- `hooks/` — cross-cutting hooks: `useFrameworkReady`, `useKeyboardNavigation`, `useToast`.
- `data/mockData.ts` — mock content for development and examples.
- `assets/images/*` — app imagery and icons.

## Conventions

- Screens import from `expo-router` (`Link`, `useRouter`, `useLocalSearchParams`).
- Layouts should compose `ScreenLayout` for consistent padding/status bar.
- Prefer `components/ui/*` for primitives. Extract new primitives if reused ≥2 screens.
- Use `useToast` for user feedback; prefer non-blocking UX.
- Keep dynamic route params typed via `useLocalSearchParams<{ id: string }>()` style.

## Patterns and utilities

- Lists: use `FlatList`/`SectionList` with stable keys; memoize renderItem when heavy.
- Loading/Empty states: reuse `Skeleton` and `EmptyState` components.
- Sharing/QR: follow patterns in `components/sharing/QRCodeGenerator.tsx` and `QRCodeScanner.tsx`; provide web fallbacks.
- Location/camera permissions: request at point of use with respective `expo-*` APIs.

## Web considerations

- Ensure components render on web; guard native-only APIs.
- Use `react-native-svg` and `react-native-webview` that are configured in `package.json`.

## Performance notes

- Memoize expensive subtrees; split components where needed.
- Avoid unnecessary re-renders in tab screens; keep state local to screens.

## Quality

- Run `expo lint` before committing; avoid suppressing TypeScript errors.
- Follow import order: node/external → `@/` alias → relative.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MelvinNunes) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
