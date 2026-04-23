## ride-today

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ride Today is a React Native/Expo app that calculates "ride scores" (0-5) based on weather conditions to help users determine if it's a good time to ride (bike/motorcycle). The app uses Weather API data and applies temperature modifiers and day/night adjustments to produce a final score.

## Commands

```bash
# Development
yarn start              # Start Expo dev server
yarn ios                # Run on iOS device
yarn android            # Run on Android

# Testing
yarn test               # Run all checks (translations, unit tests, lint, types)
yarn test:unit          # Run Jest unit tests only
yarn test:unit --watch  # Watch mode with typeahead filtering
yarn test:unit path/to/file.test.ts  # Run single test file

# Code quality
yarn test:lint          # Run ESLint
yarn test:types         # Run TypeScript type checking

# Translations (Lingui)
yarn translations:extract   # Extract new strings to .po files
yarn translations:compile   # Compile translations
yarn translations:update    # Extract and compile

# Scaffolding
yarn create:module -- <name>  # Create new module with standard folder structure
```

## Architecture

### Path Aliases

- `#app/*` → `src/app/*`
- `#modules/*` → `src/modules/*`
- `#shared/*` → `src/shared/*`
- `#design-system/*` → `src/design-system/*`
- `#testing/*` → `src/testing/*`

### Directory Structure

- `src/app/` - Expo Router file-based routes
- `src/modules/` - Feature modules (weather, rideScore, location, charts, etc.)
- `src/shared/` - Cross-cutting concerns (i18n, infra, hooks, components)
- `src/design-system/` - Theme, reusable UI components (Button, Switch, etc.)
- `src/testing/` - Jest setup and test utilities

### Module Structure

Each module in `src/modules/` follows: `components/`, `hooks/`, `api/`, `types/`, `utils/`, `domain/`

### Key Domain Logic

The ride score calculation (`src/modules/rideScore/domain/` and `src/modules/weather/domain/`):

- Weather API condition codes map to base notation (0-5) in `notationMapping.ts`
- Temperature modifiers adjust the score in `conditionScore.ts`
- Night conditions are halved

### Tech Stack

- **UI**: Emotion Native for styling, Zustand for state, React Query for data fetching
- **Navigation**: Expo Router (file-based)
- **i18n**: Lingui with PO files (en, fr locales in `src/shared/i18n/locales/`)
- **Storage**: react-native-mmkv
- **Charts**: react-native-gifted-charts

### Environment Configuration

- Stages: `dev`, `staging`, `production` (set via `STAGE` env var)
- App config in `app.config.ts` with per-stage settings
- Feature flags accessed via `src/shared/env/featureFlags.ts`

## Workflow

1. Create feature branch from `main`.
2. Write code adhering to the established architecture and patterns.
3. Commit changes with clear messages. Use conventional commits if possible, e.g., `feat: add temperature modifier logic`.
4. Run tests and linters before committing with `yarn test`, fix issues.
5. Merge to `main` with merge commits.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JulienIzz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
