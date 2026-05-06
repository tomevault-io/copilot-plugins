## tomotv

> Tomotv is an Expo Router TV app. Platform scaffolding lives in `android/`, `ios/`, and `Images.xcassets/`, while routed screens sit in `app/`. Shared UI primitives belong in `components/`, hooks in `hooks/`, and global state in `contexts/`. Playback and catalog clients live in `services/`; helpers and types stay in `utils/` and `types/`. Tests mirror their targets (e.g., `services/__tests__/libraryManager.test.ts`). Media assets live in `assets/`, and docs in `docs/`.

# Repository Guidelines

## Project Structure & Module Organization

Tomotv is an Expo Router TV app. Platform scaffolding lives in `android/`, `ios/`, and `Images.xcassets/`, while routed screens sit in `app/`. Shared UI primitives belong in `components/`, hooks in `hooks/`, and global state in `contexts/`. Playback and catalog clients live in `services/`; helpers and types stay in `utils/` and `types/`. Tests mirror their targets (e.g., `services/__tests__/libraryManager.test.ts`). Media assets live in `assets/`, and docs in `docs/`.

## Build, Test, and Development Commands

Install dependencies with `npm install`. `npm run start` launches the Expo dev server. `npm run android` / `npm run ios` build and deploy to the respective simulators. Run `npm run lint` (ESLint + Prettier autofix) before pushing. Execute `npm run test`, `npm run test:watch`, or `npm run test:coverage` to validate logic, and use `npm run prebuild` or `npm run prebuild:tv` (sets `EXPO_TV=1`) when regenerating native projects.

## Coding Style & Naming Conventions

The codebase is TypeScript-first with strict ESLint and Prettier configs—use 2-space indentation, semicolons, and single quotes inside TS/TSX. Components and hooks follow `PascalCase` filenames (`VideoShelf.tsx`, `usePlayback.ts`). Utilities and services use `camelCase`. Keep styles beside components, prefer `StyleSheet.create`, and avoid editing generated outputs inside `android/` or `ios/` unless performing a native patch.

## Testing Guidelines

Jest (via `jest-expo`) drives the suite. Place specs in local `__tests__` folders and suffix files with `.test.ts(x)` or `.threading.test.ts(x)` for concurrency helpers. Mock network I/O within services, lean on `react-test-renderer` harnesses for hooks/contexts (RTL is not wired up here), and aim for ≥80% statement coverage when running `npm run test:coverage`. Every bugfix should ship with a regression test.

## Commit & Pull Request Guidelines

Follow the existing `type: concise summary` format (e.g., `fix: clear player queue`) and keep commits scope-limited. Reference issue IDs when applicable and bundle related asset/config updates with the code. Pull requests should include: a short purpose statement, testing steps (commands + expected outcome), screenshots or recordings for UI changes, and any follow-up tasks. Request reviews from platform owners when touching `services/` or device-specific modules.

## Security & Configuration Tips

Never commit secrets; rely on secure store APIs and Expo config values. Configure your Jellyfin server URL in `.env.local` for development. For TV builds, set `EXPO_TV=1` locally and verify the Apple/Android TV asset sets (`TVOS_ICONS.md`, `Images.xcassets/`) stay in sync with feature work.

---
> Source: [keiver/tomotv](https://github.com/keiver/tomotv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
