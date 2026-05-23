## familyforagingv3

> This repository is an Expo + React Native app written in TypeScript with NativeWind/Tailwind, React Navigation, and Zustand for state. Use this guide to stay consistent and productive.

# Repository Guidelines

This repository is an Expo + React Native app written in TypeScript with NativeWind/Tailwind, React Navigation, and Zustand for state. Use this guide to stay consistent and productive.

## Project Structure & Modules
- `App.tsx`/`index.ts`: app entry and registration.
- `src/`: primary code
  - `api/`: AI, media, and network helpers
  - `components/`: reusable UI (PascalCase files)
  - `screens/`: navigation screens (PascalCase)
  - `navigation/`: navigator setup
  - `state/`: Zustand stores
  - `utils/`, `types/`, `data/`: helpers, typings, seed data
- `assets/`: images, fonts. Configure in `app.json` if adding.

## Build, Test, and Development
- Install: `npm install` (Node 18+). Bun lock is present but `npm` is the source of truth.
- Start (Expo Dev): `npm run start`
- Run Android: `npm run android`
- Run iOS: `npm run ios`
- Web preview: `npm run web`
- EAS builds: `npx eas build -p ios|android` (see `eas.json`).

## Coding Style & Naming
- Language: TypeScript. Prefer explicit types at module boundaries.
- Formatting: Prettier enforced via ESLint. Settings: `tabWidth: 2`, `printWidth: 120`, `singleQuote: false` (use double quotes).
- Linting: `eslint` with `expo` config; Prettier errors enabled. Run in editor or `npx eslint .`.
- Naming: files in `components/` and `screens/` use PascalCase; utilities/data/types use `kebab-case`/`camelCase` as existing. React components: PascalCase. Zustand store files: `*-store.ts`.
- Styles: Tailwind via NativeWind `className` strings; prefer composed utility classes over inline styles.

## Testing Guidelines
- Runner: Jest with `jest-expo` preset. Setup in `jest.config.js` and `jest.setup.ts`.
- Commands: `npm test` (run all), `npm run test:watch`, `npm run test:ci`.
- Locations: colocate as `*.test.ts(x)` or under `__tests__/` (examples added).
- Libraries: `@testing-library/react-native` and `@testing-library/jest-native` for assertions.
- Scope: unit tests for `src/utils/`; render tests for `components/`/`screens/` with minimal mocks.

## Commit & Pull Requests
- Commits: use concise, imperative subjects (≤72 chars). Example: `Fix map startup crash on Android 14`.
- Reference issues in body when applicable. Group logical changes per commit.
- PRs: include summary, screenshots for UI, repro/verification steps, and linked issues. Keep PRs scoped; update docs if structure or commands change.

## Security & Configuration
- Secrets: never commit `.env` values; prefer Expo SecureStore for runtime secrets.
- Patches: this repo uses `patch-package` (see `patchedDependencies`). Keep patches small and documented in the PR.

---
> Source: [Kieransaunders/familyforagingv3](https://github.com/Kieransaunders/familyforagingv3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
