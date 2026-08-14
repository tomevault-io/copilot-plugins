## explorasisejarahdbp

> - Source lives in `app/`, `components/`, `contexts/`, `data/`, `types/`, `utils/`, and assets in `assets/`.

# Repository Guidelines

## Project Structure & Module Organization
- Source lives in `app/`, `components/`, `contexts/`, `data/`, `types/`, `utils/`, and assets in `assets/`.
- Expo Router organizes screens in `app/(game)` and `app/(tabs)`; file names define routes. Avoid renaming routes casually.
- Question content sits in `data/questions/`; shared UI in `components/game/`; global state in `contexts/GameContext.tsx`.
- TypeScript path alias `@/` maps to repo root (see `tsconfig.json`).

## Build, Test, and Development Commands
- `yarn start` — Start the Expo dev server.
- `yarn ios` / `yarn android` / `yarn web` — Launch on platform.
- `yarn lint` — Run ESLint (expo config).
- `node scripts/reset-project.js` — Clean/reset helper.
- `npx tsc --noEmit` — Type-check the project.
- Tip: If Metro cache misbehaves, run `npx expo start --clear`.

## Coding Style & Naming Conventions
- Language: TypeScript (strict). Indentation: 2 spaces. Semicolons: required. Quotes: single.
- Components: PascalCase (`StatusBar.tsx`), hooks/utilities: camelCase, files: kebab-case where applicable (`fill-blank-question.tsx`).
- Prefer `@/` imports over long relatives. Keep components small and focused.
- Linting: `eslint-config-expo`; fix warnings before merging. No Prettier configured—match existing style.

## Testing Guidelines
- No automated tests yet. Follow `TESTING_GUIDE.md` for manual QA flows (tutorial, quiz types, success modal, persistence, responsiveness, haptics, animations).
- Before opening a PR: run `yarn lint` and `npx tsc --noEmit` and verify key flows on at least one device + one simulator.
- If adding tests, prefer React Native Testing Library + Jest; place near sources under `__tests__/` and name files `*.test.tsx`.

## Commit & Pull Request Guidelines
- Use Conventional Commits: `feat:`, `fix:`, `docs:`, `refactor:`, `chore:`.
- PRs should include: clear summary, screenshots/screen recordings for UI changes, reproduction and test steps, and linked issues.
- Keep PRs focused and small. Ensure no TypeScript or ESLint errors.

## Security & Configuration Tips
- Do not commit secrets or tokens. Use runtime storage via `expo-secure-store` and environment config in `app.json` when needed.
- Only add assets you have rights to distribute (see `assets/images/`).

## Agent-Specific Instructions
- Respect Expo Router: changing files under `app/` changes routes. Avoid broad renames or restructures.
- Keep changes minimal, align with existing patterns, and update docs when modifying behavior or structure.

---
> Source: [amirariff91/explorasisejarahdbp](https://github.com/amirariff91/explorasisejarahdbp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
