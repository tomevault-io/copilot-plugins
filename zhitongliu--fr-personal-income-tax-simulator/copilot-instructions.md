## fr-personal-income-tax-simulator

> - `src/` — TypeScript/React source.

# Repository Guidelines

## Project Structure

- `src/` — TypeScript/React source.
  - `src/components/` — UI components (forms, results, cards).
  - `src/services/taxCalculator.ts` — tax computation logic.
  - `src/data/taxBrackets.ts` — tax brackets by year.
  - `src/types/` and `src/utils/` — shared types/utilities.
- `public/` — static assets served as-is.
- `dist/` — production build output (generated).
- `.github/workflows/deploy.yml` — GitHub Pages deployment on pushes to `main`.

## Build, Test, and Development Commands

Use Node.js (CI uses Node 20).

- `npm install` — install dependencies.
- `npm run dev` — start Vite dev server (default: `http://localhost:5173`).
- `npm run build` — typecheck + build production assets into `dist/`.
- `npm run preview` — serve the built app locally from `dist/`.
- `npm run lint` — run ESLint on the repo.
- `npm run deploy` — build then publish `dist/` via `gh-pages` (optional; CI deploys via Actions).

## Coding Style & Naming Conventions

- Language: TypeScript + React (Vite).
- Formatting: follow existing style (2-space indentation; single quotes; trailing commas where present).
- Components: `PascalCase.tsx` (e.g., `TaxResults.tsx`), exported as `PascalCase`.
- Utilities: `camelCase.ts` (e.g., `formatCurrency.ts`).
- Keep calculation logic in `src/services/` (avoid embedding business rules in UI components).

## Testing Guidelines

No dedicated test runner is currently configured. If you add tests, prefer colocating them near code (e.g., `src/services/taxCalculator.test.ts`) and keep them deterministic (no locale/time dependencies).

## Commit & Pull Request Guidelines

Recent history follows lightweight Conventional Commits-style prefixes (e.g., `fix: ...`, `refactor: ...`). Keep messages short, imperative, and scoped.

For PRs:
- Describe the user-visible behavior change and link any issue.
- Include screenshots/GIFs for UI changes.
- If you modify brackets/years, call out the affected year(s) and files (typically `src/data/taxBrackets.ts` and `src/services/taxCalculator.ts`).

## Security & Configuration Notes

If deploying to GitHub Pages from a repo name other than `tax_simulator`, update `base` in `vite.config.js` (e.g., `base: '/your-repo-name/'`).

---
> Source: [zhitongLIU/fr_personal_income_tax_simulator](https://github.com/zhitongLIU/fr_personal_income_tax_simulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
