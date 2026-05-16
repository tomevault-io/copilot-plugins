## react-vocabulary

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
npm start                # Expo dev server (press w for web, a for Android, i for iOS)
npm run web              # Run directly in browser
npm test                 # Run all Jest tests
npm test -- --testPathPattern='answerValidator'  # Run a single test file by name
npm test -- --coverage   # Run with coverage report
npm run type-check       # TypeScript strict mode check
npm run lint             # ESLint check
npm run lint:fix         # Auto-fix lint issues
npm run format           # Prettier format all files
npm run check            # All checks: type-check + lint + test
```

## Architecture

**Expo Router + React Native** cross-platform app (iOS, Android, Web) with file-based routing.

### Routing Layer (`/app/`)

Screens in `/app/` are thin wrappers that render feature screens from `/src/features/`. The root layout (`_layout.tsx`) handles theme provider setup, store hydration, and cloud sync orchestration.

### Feature Modules (`/src/features/`)

Each feature (vocabulary, quiz, progress, settings, onboarding, help) is self-contained with co-located `screens/`, `components/`, and `utils/` directories.

### Shared Layer (`/src/shared/`)

- **`store/`** - Zustand stores: `vocabularyStore`, `quizStore`, `progressStore`, `settingsStore`, `adaptiveDifficultyStore`. Stores use debounced AsyncStorage persistence (500ms). Cloud sync is throttled to 5-minute intervals.
- **`types/`** - TypeScript definitions for vocabulary, quiz, progress, navigation
- **`ui/`** - Reusable components (Button, Card, Typography, ProgressBar, etc.)
- **`hooks/`** - `useSound` (expo-av), `useHaptics` (expo-haptics), `useReducedMotion`
- **`services/`** - Cloud sync API client with retry logic (3 attempts)
- **`lib/`** - Theme (Material Design 3 light/dark), AsyncStorage wrapper, Levenshtein distance

### Path Aliases (tsconfig.json)

`@/*` maps to `./src/*`, with sub-aliases for `@/features/*`, `@/shared/*`, `@/assets/*`.

### Quiz Domain Logic

- All words from the selected level are quizzed, each gets 2 questions (1 multiple-choice + 1 fill-in-blank)
- Multiple-choice uses 3 random distractors from the same difficulty level
- Fill-in-blank uses Levenshtein distance for typo tolerance (1 char)
- Word progress: 0 (not started) → 1 (seen) → 2 (partial) → 3 (mastered)

### Vocabulary Data

18 themed JSON word lists in `/src/assets/vocabulary/` with 5 difficulty levels (Basic → Professional).

### Backend

AWS SAM Lambda API in `/backend/` (separate deployment). Optional cloud sync via `EXPO_PUBLIC_SYNC_API_URL` env var.

## Testing

- **Preset**: `jest-expo` with mocks for AsyncStorage, expo-av, expo-haptics, react-native-chart-kit in `jest.setup.js`
- **Coverage**: collected from `src/**/*.{ts,tsx}`, excludes `.d.ts` and `__tests__/`
- **Backend tests** are excluded via `testPathIgnorePatterns`
- Tests co-located in `__tests__/` directories next to source files

## UI Framework

React Native Paper (Material Design 3) with light/dark/auto theme support defined in `src/shared/lib/theme.ts`.

---
> Source: [HatmanStack/react-vocabulary](https://github.com/HatmanStack/react-vocabulary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
