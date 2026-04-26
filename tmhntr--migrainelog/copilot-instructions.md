## migrainelog

> MigraineLog is a React Native (Expo) mobile app for tracking migraine triggers, episodes, and treatments. All data is stored locally on-device using SQLite. There is no backend server.

# CLAUDE.md

## Project Overview

MigraineLog is a React Native (Expo) mobile app for tracking migraine triggers, episodes, and treatments. All data is stored locally on-device using SQLite. There is no backend server.

## Tech Stack

- React Native with Expo (managed workflow, eject to bare if widget support requires it)
- TypeScript (strict mode)
- SQLite via `expo-sqlite` for local persistence
- Zustand for state management
- React Navigation for routing (bottom tabs + stack navigators)

## Architecture Decisions

- **Local-first:** All data lives in SQLite on the device. No network calls, no auth, no cloud sync.
- **Widget support:** Home screen widgets for quick logging and risk display. This may require ejecting from Expo managed workflow or using expo-dev-client with native modules.
- **Risk calculation:** The "current risk level" is computed from recent triggers and historical episode frequency. The algorithm lives in `src/utils/risk.ts` and should be pure/testable.

## Code Conventions

- Use functional components with hooks. No class components.
- Prefer named exports over default exports.
- File naming: `kebab-case.ts` for utilities, `PascalCase.tsx` for components/screens.
- Keep database access in `src/db/` — screens and components should not import `expo-sqlite` directly.
- Types go in `src/models/` — one file per entity (trigger.ts, episode.ts, treatment.ts) plus a shared `event.ts` for common fields.

## Data Model

Three event types share common fields (id, timestamp, notes):

- **Trigger** — category (enum: sleep, stress, food, weather, hormonal, other), severity (1-5)
- **Episode** — severity (1-10), duration_minutes, symptoms (JSON array), aura (boolean)
- **Treatment** — type (enum: medication, rest, hydration, caffeine, other), name (free text), effective (boolean, nullable — set after the fact)

## Key Commands

```bash
npm install --legacy-peer-deps  # Install dependencies (--legacy-peer-deps required due to react-test-renderer peer conflict)
npx expo start       # Start dev server
npx expo run:ios     # Run on iOS
npx expo run:android # Run on Android
npm test             # Run tests (Jest)
npm run lint         # Run ESLint
npm run typecheck    # Run tsc --noEmit
```

## Testing

- Use Jest with React Native Testing Library.
- Test risk calculation logic thoroughly — it is the core business logic.
- DB query helpers should have integration tests using an in-memory SQLite database.

## Important Patterns

- All DB operations are async. Use the query helpers in `src/db/queries.ts` which return typed results.
- Zustand stores in `src/stores/` hydrate from SQLite on app launch and write-through on mutations.
- The widget reads from a shared SQLite database or shared UserDefaults/SharedPreferences for the risk level value.
- Navigation structure: bottom tabs for Dashboard, Triggers, Episodes, Treatments, Settings. Each tab has its own stack navigator.

## System Design

The full architecture document is at [`docs/SYSTEM_DESIGN.md`](docs/SYSTEM_DESIGN.md). It covers database schema, state management, navigation, risk calculation, widget architecture, component hierarchy, and the complete file manifest.

## Claude Skills

The following custom skills are available in `.claude/skills/`:

| Skill | Description |
|-------|-------------|
| `/generate-screen` | Scaffold list, detail, form, or custom screen with navigation integration |
| `/generate-component` | Create reusable UI component with props interface and tests |
| `/generate-migration` | Add numbered DB migration with schema changes + type/query updates |
| `/generate-test` | Generate test file for any module (component, store, util, query) |
| `/add-event-type` | Add new event type or extend existing one across all layers |
| `/generate-store` | Create Zustand store with optional SQLite persistence |
| `/dev-check` | Run typecheck, lint, test, and expo doctor in sequence |
| `/generate-query` | Add typed query helper to `src/db/queries.ts` with tests |
| `/scaffold-feature` | End-to-end feature scaffolding orchestrating other skills |
| `/generate-stats` | Add statistical/analytics function with dashboard integration |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tmhntr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
