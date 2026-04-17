## meritable

> Meritable is a React Native habit tracking app built with Expo and TypeScript. It uses file-based routing (Expo Router) and runs on iOS, Android, and Web.

# Meritable - Codebase Structure and Rules

## Project Overview

Meritable is a React Native habit tracking app built with Expo and TypeScript. It uses file-based routing (Expo Router) and runs on iOS, Android, and Web.

## Tech Stack

- Expo ~54, React Native 0.81, Expo Router, NativeWind (Tailwind), React Query, Supabase (auth + cloud), Dexie / AsyncStorage (local), Jest + React Native Testing Library, EAS Build

## Directory Map

- `app/` — Routes and layouts (see .cursor/rules when editing)
- `components/` — Reusable UI: `common/`, `habits/list/`, `habits/detail/`, `quick-wins/list/`, `quick-wins/detail/`, `data/`, `Modals/`, `workouts/` (see .cursor/rules when editing)
- `db/` — Data layer and auth (see .cursor/rules when editing)
- `lib/` — Utils (Colors, dateUtils, useWindowWidth)
- `assets/` — Fonts, images
- `ios/` — iOS native

## Conventions (global)

- Path alias `@/*`; TypeScript strict; NativeWind for styling. New features: routes in app/, components in components/, utilities in lib/. Support iOS, Android, and Web.

## Development Guidelines

1. New features: add routes in `app/`, components in `components/`, utilities in `lib/`.
2. Database changes: update `HabitDatabaseInterface` in `db/habitDatabase.ts` and all three implementations (dexieDb, asyncStorageDb, supabaseDb).
3. Styling: NativeWind classes; theme colors from `lib/Colors.ts`.
4. Tests: `__tests__/` next to code; mocks in `__mocks__/` (e.g. `db/__mocks__/`).
5. When you add or remove routes, top-level dirs, or major modules, update this file and the matching `.cursor/rules/*.mdc` so the rules stay in sync with the codebase.

## Build & Deployment

- iOS: EAS Build (development, production, simulator). Web: `expo export --platform web`. Config: `eas.json`, `app.json`.

## Important Notes

- New architecture enabled; typed routes (Expo Router); dark mode UI; React Query for async data; drag-and-drop reordering via @dnd-kit.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dtom90) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
