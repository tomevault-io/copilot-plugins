## loyus

> Offline-first loyalty-card wallet for iOS/Android. Expo SDK 55, React Native 0.83, React 19, TypeScript strict.

# Loyus

Offline-first loyalty-card wallet for iOS/Android. Expo SDK 55, React Native 0.83, React 19, TypeScript strict.

## Build and Run

```bash
pnpm ios              # Dev build for iOS (no Expo Go)
pnpm android          # Dev build for Android
pnpm check:fix        # Biome lint + format (biome check --write)
pnpm typecheck        # TypeScript strict (tsc --noEmit)
pnpm test             # Jest unit tests
pnpm test:e2e         # Maestro E2E flows
```

## Architecture (4-layer, strict boundaries)

| Layer  | Path          | Allowed Imports                         |
|--------|---------------|-----------------------------------------|
| Domain | `src/domain/` | stdlib only (zero RN/Expo imports)      |
| State  | `src/state/`  | domain, infra/persistence               |
| Infra  | `src/infra/`  | domain, expo/RN native modules          |
| UI     | `src/ui/`, `app/` | state (via selectors), domain types |

- Components are pure presentational -- no business logic, no direct persistence
- Logic lives in Zustand stores (one per bounded concern)
- Store subscriptions via selectors only (never whole-store)
- Domain layer enforced by `src/domain/__tests__/architecture.test.ts`

## Linting (Biome.js, NOT ESLint)

- Run: `pnpm check:fix` (biome check --write)
- Config: `biome.json` at repo root
- NEVER suggest ESLint, Prettier, or eslint-* packages

## Conventions

- TypeScript strict: `strict: true`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`
- No `any` in app code (noExplicitAny: error in Biome)
- Optional fields: `field?: T | undefined`
- Spread conditionally: `...(val !== undefined ? { field: val } : {})`
- Branded types for IDs: `CardId = string & { readonly [Brand]: typeof Brand }`
- Readonly interfaces for domain types
- `import type` for type-only imports (enforced by Biome useImportType)

## i18n

- All user-facing strings wrapped in `t('section.key')` via react-i18next
- Locale files in `src/infra/i18n/locales/` (en, fr, es, pt, ru, de)
- Synchronous init (`initImmediate: false`) -- no async flash
- When adding a string: add key to ALL 6 locale files

## Styling (Unistyles 3.x)

- Theme tokens in `src/ui/theme/tokens.ts`
- `StyleSheet.configure()` in `src/ui/theme/unistyles.ts`
- Import `{ StyleSheet, useUnistyles }` from `src/ui/theme/unistyles` (re-exports)
- Light/dark theme with `adaptiveThemes: true`
- Font: Manrope (Regular, Medium, SemiBold, Bold, ExtraBold)
- All colors from `theme.colors`, all spacing from `theme.spacing` -- never hardcoded
- Exception: `#2D4739` in barcode detail (always dark green regardless of theme)

## State (Zustand 5 + MMKV)

- Stores: cardStore, settingsStore, uiStore, captureStore
- MMKV persistence via `createJSONStorage(() => mmkvStateStorage)`
- `partialize` to exclude actions from persistence
- Schema versioning with `version` + `migrate` callback using domain `runMigrations`
- `expo-crypto` `randomUUID()` for ID generation (not Math.random or uuid)

## Testing

- Jest 29 + jest-expo (NOT Jest 30 -- breaking import scope changes)
- `jest.config.js` (not .ts, avoids ts-node dependency)
- No snapshot tests -- use RNTL assertions
- Domain: >= 80% coverage, zero RN imports
- Mocks in `__mocks__/` for native modules (unistyles, mmkv, expo-crypto, nitro-modules)
- Maestro E2E in `.maestro/` (shared flows in `.maestro/shared/`)

## Commit Style

- Conventional commits: `type(scope): message`
- Types: feat, fix, test, docs, refactor, chore
- Scope = phase number or feature area

---
> Source: [Loyus-App/Loyus](https://github.com/Loyus-App/Loyus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
