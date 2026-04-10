## la-pape

> This repository is an Expo Router React Native app written in TypeScript.

# Repository Guidelines

## Project Structure & Module Organization
This repository is an Expo Router React Native app written in TypeScript.
- `app/`: route-based screens (`(tabs)/`, `(auth)/`, dynamic routes like `producto/[id]`).
- `components/`: reusable UI and feature components (e.g., `components/ventas`, `components/forms`).
- `lib/`: integrations and business logic (Firebase setup, services, validations, PDF helpers).
- `store/`: Zustand state stores.
- `interface/`: shared domain types/interfaces.
- `assets/`: static images and icons.
- `android/`, `ios/`: native projects generated for platform builds.

Use the `@/*` path alias from `tsconfig.json` for internal imports.

## Build, Test, and Development Commands
- `npm install`: install dependencies.
- `npm run start`: start Expo dev server.
- `npm run android`: run on Android device/emulator.
- `npm run ios`: run on iOS simulator (macOS required).
- `npm run web`: run web target locally.
- `npm run lint`: run ESLint via Expo config.

Example workflow:
```bash
npm run lint
npm run start
```

## Coding Style & Naming Conventions
- Language: TypeScript (`strict` mode enabled).
- Linting: `eslint-config-expo` (`eslint.config.js`).
- Indentation: 2 spaces; keep formatting consistent with surrounding files.
- Components/screens: `PascalCase` filenames (`ProductCardMobile.tsx`).
- Hooks: `use-*` naming (`use-product-search.ts`).
- Stores/services/interfaces: kebab-case directories/files with descriptive names.

Prefer small, focused modules and colocate feature UI under `components/<feature>/`.

## Testing Guidelines
No automated test framework is currently configured in `package.json`.
- Minimum validation before PR: `npm run lint` and manual verification on at least one platform (`android`, `ios`, or `web`).
- If adding tests, prefer colocated `*.test.ts(x)` files and document the run command in `package.json`.

## Commit & Pull Request Guidelines
Recent history shows short, imperative commit subjects, often in Spanish (e.g., `modificando git ignore...`, `Update dependencies...`).
- Keep commit messages concise, present tense, and scoped to one change.
- PRs should include: purpose summary, key files changed, manual test steps, and screenshots/videos for UI changes.
- Link related issue/task IDs when available and note any env/config updates (`.env.example`, Firebase settings).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/francost15)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/francost15)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
