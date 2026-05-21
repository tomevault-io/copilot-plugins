## quangpham-dev

> The `app/` directory hosts the Next.js App Router plus route-level layouts, while shared UI lives in `components/` and reusable hooks in `hooks/`. Keep configuration (feature flags, i18n, SEO metadata) inside `config/` and `constants/`, and place data helpers or API wrappers under `lib/`. Static inputs such as icons or fonts belong in `assets/`, whereas runtime-servable files go to `public/`. Unit tests sit in `test/` and may co-locate with features using `*.spec.tsx`, and cross-browser Playwright suites live in `e2e/`.

# Repository Guidelines

## Project Structure & Module Organization
The `app/` directory hosts the Next.js App Router plus route-level layouts, while shared UI lives in `components/` and reusable hooks in `hooks/`. Keep configuration (feature flags, i18n, SEO metadata) inside `config/` and `constants/`, and place data helpers or API wrappers under `lib/`. Static inputs such as icons or fonts belong in `assets/`, whereas runtime-servable files go to `public/`. Unit tests sit in `test/` and may co-locate with features using `*.spec.tsx`, and cross-browser Playwright suites live in `e2e/`.

## Build, Test, and Development Commands
Use Bun for every script:
- `bun dev` launches the Turbopack dev server, and `bun storybook` isolates components locally.
- `bun build` compiles the production bundle; follow with `bun start` to smoke-test it.
- `bun check` (Biome lint + format) and `bun check:fix` resolve style issues; `bun check:unused` prunes dead code through Knip.
- `bun test`, `bun test:coverage`, and `bun test:ui` run Vitest in CLI, coverage, or watch-UI mode via happy-dom/jsdom.
- `bun e2e`, `bun e2e:ui`, and `bun e2e:debug` run Playwright at different verbosity levels; `bun analyze` and `bun lighthouse` spot bundle/perf regressions before release.

## Coding Style & Naming Conventions
TypeScript is required across `app/` and `components/`; prefer server components unless a hook forces a client boundary. Biome enforces formatting (80-character max line width, `asNeeded` semicolons, `es5` trailing commas) and bans unused imports/variables, so rely on `bun check:fix` before committing. Name React components and files with `PascalCase`, hooks with the `useFoo` prefix, and keep directories kebab-case (`dark-section/hero.tsx`). Consolidate tailwind classes with `clsx`/`cva` helpers in `lib/utils.ts`.

## Testing Guidelines
Author Vitest specs next to the feature or under `test/` using `*.spec.ts`/`*.spec.tsx`, and use `@testing-library/react` plus `happy-dom` for DOM assertions. Keep unit tests deterministic by mocking network clients within `lib/__mocks__`. E2E flows use Playwright specs in `e2e/`; prefer descriptive names (`resume-download.e2e.ts`) and leverage `playwright.config.ts` fixtures. Run `bun test` before committing and `bun test:coverage` plus `bun e2e` before opening a PR; CI gates on these commands and Lefthook reruns them on `pre-push`.

## Commit & Pull Request Guidelines
Commits follow Conventional Commits (enforced by `commitlint.config.js`), so write `type(scope): subject` entries such as `feat(i18n): add german copy`. Run `bunx cz` (Commitizen) if you prefer a guided prompt. Lefthook executes `bun check --fix` and `bun test --run` on `pre-commit`, so keep the hook installed and let it stage any auto-fixes. Pull requests should include a crisp summary, linked GitHub issue (or “N/A”), screenshots/GIFs for visible UI, and notes on verification commands. Ensure `.env.example` covers every new secret, update Storybook stories when UI changes, and mention lighthouse/analysis results when performance is at stake.

## Environment & Configuration Tips
Duplicate `.env.example` to `.env.local` for local secrets and avoid checking credentials into git. Update localization files in `i18n/` and sync corresponding enums in `constants/locales.ts`. When adding assets, compress them and import through `next/image` for optimized delivery. For feature toggles, expose a single source of truth in `config/features.ts` and surface flags through server components to keep hydration payloads lean.

---
> Source: [quang-pham-dev/quangpham.dev](https://github.com/quang-pham-dev/quangpham.dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
