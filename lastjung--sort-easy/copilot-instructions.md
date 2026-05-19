## sort-easy

> - `src/` holds all React source code.

# Repository Guidelines

## Project Structure & Module Organization
- `src/` holds all React source code.
- `src/components/` contains UI building blocks (dashboard, controls, cards).
- `src/algorithms/` implements sorting algorithms; `src/algorithms/index.js` exports the catalog.
- `src/constants/` stores shared constants (e.g., color tokens).
- `public/` contains static assets served by Vite.
- `docs/` includes project notes and algorithm documentation (see `docs/sorting_logic.md`).
- `dist/` is the Vite build output (generated, do not edit).

## Build, Test, and Development Commands
- `pnpm dev`: start the Vite dev server with hot reload.
- `pnpm build`: create a production build in `dist/`.
- `pnpm preview`: serve the production build locally.
- `pnpm lint`: run ESLint across the repo.

## Coding Style & Naming Conventions
- Language: React (JSX) with ESM imports.
- Indentation: 2 spaces, semicolons required (see `src/App.jsx`).
- Components use `PascalCase` filenames (e.g., `SortControls.jsx`).
- Utility modules and constants use `camelCase`/`kebab-case` as appropriate (`src/constants/colors.js`).
- Linting: ESLint with `eslint-plugin-react-hooks` and `eslint-plugin-react-refresh`; fix lint errors before pushing.

## Testing Guidelines
- No automated test framework is currently configured.
- If adding tests, align with Vite + React defaults and document the command in `package.json`.
- Prefer colocated tests near the module under test (e.g., `src/components/SortCard.test.jsx`).

## Commit & Pull Request Guidelines
- Commit messages follow a Conventional Commits style (e.g., `fix: ...`, `docs: ...`).
- Keep commits focused and descriptive; avoid mixing refactors with feature changes.
- PRs should include:
  - A brief summary of changes and rationale.
  - Screenshots or short clips for UI/visual updates.
  - Notes on any algorithm behavior changes or performance impacts.

## Security & Configuration Tips
- This repo ships a client-only Vite app; avoid committing secrets to the frontend.
- If introducing environment variables, document them and add a `.env.example`.

---
> Source: [lastjung/sort-easy](https://github.com/lastjung/sort-easy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
