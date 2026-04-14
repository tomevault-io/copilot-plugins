## footalert-mobile

> FootAlert is a multi-platform (iOS, Android, Web, Desktop) football scores application. It provides real-time match data, standings, and player/team statistics using the API-FOOTBALL service.

# GEMINI.md - FootAlert Project Context

## Project Overview
FootAlert is a multi-platform (iOS, Android, Web, Desktop) football scores application. It provides real-time match data, standings, and player/team statistics using the API-FOOTBALL service. 

- **Primary Platform:** React Native (iOS/Android)
- **Secondary Platforms:** Web (React/Vite) and Desktop (Tauri 2 shell)
- **Backend:** Node.js (Fastify) BFF (Backend-For-Frontend) to secure API keys and normalize data.
- **Architecture:** Feature-first organization, offline-first with TanStack Query persistence, and a shared core for cross-platform parity.

## Core Mandates & Technical Standards

### 1. Data Flow & Security
- **Strict BFF Proxy:** The mobile application MUST NOT call API-Football directly. All requests go through the BFF (`footalert-bff`).
- **Secrets Management:** `API_FOOTBALL_KEY` and other sensitive keys must only exist on the server (BFF). Never embed them in the mobile app.
- **Offline-First:** Use TanStack Query with `AsyncStorage` persistence. UI should display cached data immediately while revalidating in the background.

### 2. UI & Development Conventions
- **Internationalization (i18n):** NO HARDCODED STRINGS in UI components. Use `t()` from `react-i18next` and manage keys in `src/ui/shared/i18n/locales`.
- **File Size Limits:** 
  - Mobile UI files (`src/ui/**/*.tsx`) must not exceed **350 lines**.
  - BFF Route files (`footalert-bff/src/routes/**/*.ts`) must not exceed **500 lines**.
- **Typing:** Strict TypeScript throughout. Navigation must be strictly typed (`src/ui/app/navigation/types.ts`).
- **Performance:** 
  - Use `FlashList` for lists.
  - Hermes engine must remain enabled.
  - Implement lazy loading for screens to optimize cold start.
- **Refactoring:** When modifying Teams or Players features, follow the modular structure in `src/ui/features/teams` and `src/ui/features/players`.

### 3. API Contract
- The API contract is defined in `packages/api-contract/openapi/footalert.v1.yaml`.
- Types are generated in `packages/api-contract/generated/types.ts`.
- Run `npm run contract:check` before making API-related changes.

## Commands & Workflow

### Development Setup
1. **Root Install:** `npm install`
2. **BFF Setup:** `cd footalert-bff && cp .env.example .env && npm install`
3. **iOS Setup:** `bundle install && bundle exec pod install` (macOS only)

### Running the Apps
- **BFF:** `cd footalert-bff && npm run dev` (Port 3001)
- **Metro Bundler:** `npm start`
- **Android:** `npm run android`
- **iOS:** `npm run ios`
- **Web:** `npm run web:dev`

### Testing & Validation
- **All Checks:** `npm run check:all` (Lints, types, tests, file sizes, contracts)
- **Unit Tests:** `npm test`
- **Performance Audit:** `npm run perf:android:audit`
- **Storybook:** `npm run storybook`

## Key Directories
- `src/ui/features/`: Feature-specific UI components and hooks.
- `src/data/`: Endpoints, mappers, and storage logic for the mobile app.
- `footalert-bff/src/routes/`: BFF endpoint implementations.
- `packages/app-core/`: Shared business logic and services.
- `docs/`: Comprehensive architectural and performance documentation.

## Success Metrics
- **Performance:** Cold start p50 < 750ms, Janky frames < 6%.
- **Quality:** 100% i18n coverage, 0 direct 3rd party API calls from mobile.
- **Business:** CPM > $2, D7 Retention > 35%.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/romarich57)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/romarich57)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
