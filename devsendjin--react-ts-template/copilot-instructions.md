## react-ts-template

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

Package manager is **pnpm** (see `pnpm-lock.yaml`). Node version is pinned in `.nvmrc` (v24.19.0).

- `pnpm dev` — start the Vite dev server
- `pnpm build` — type-check via project references (`tsc -b`) then production build
- `pnpm preview` — preview the production build locally
- `pnpm lint:tsc` — type-check both the app (`tsconfig.app.json`) and the Vite config (`tsconfig.node.json`); run this after any change
- `pnpm tsc:dev:watch` — watch-mode type-check for `src` only
- `pnpm format` — Prettier over `src/**/*.{ts,tsx,json,scss,css}`, `package.json`, and root `*.{js,ts,html}`

There is no test runner and no ESLint config in this repo — `lint:tsc` (tsc) and Prettier are the only checks. Don't assume a `test` script or lint rules exist beyond these.

## Architecture

This is a Vite + React 19 + TypeScript template establishing folder/import conventions for apps built on top of it; most feature directories are still scaffolds (`.gitkeep`).

**Path aliases**: `components/*`, `utils/*`, `hooks/*`, `assets/*`, `context/*`, `services/*` all resolve to `src/<name>/*`. They are declared in **two places that must stay in sync**: `tsconfig.app.json` (`compilerOptions.paths`) and `vite.config.ts` (`resolve.alias`).

**Component convention**: each component lives in its own folder with three co-located files:
- `ComponentName.tsx` — implementation
- `ComponentName.module.scss` — CSS Module with its styles (Sass via `sass-embedded`)
- `index.tsx` — barrel re-export (`export * from "./ComponentName"`), so consumers import from the folder (e.g. `components/entities/Timer`) rather than the file

`src/components` is split into:
- `App/` — the root app component
- `entities/` — feature-level components composed from smaller pieces (e.g. `Timer`)
- `ui/` — reusable presentational components (currently empty scaffold)

**Styling**: global styles enter through `src/index.scss`, which `@use`s `assets/styles/reset.scss` and `assets/styles/abstracts.scss` (fonts, mixins) and wraps global rules in `@layer base`. Per-component styling uses CSS Modules, not global classes or inline styles.

**Date/time handling**: always go through `src/utils/datetime.utils.ts` instead of importing `dayjs` directly elsewhere. It centralizes:
- `Time` — millisecond constants (`Second`, `Minute`, `Hour`, `Day`, `Week`)
- `DateFormats` — named dayjs format strings (`ISO_DATE`, `ISO_DATETIME`, `ISO_TIME`, `DISPLAY_DATE`)
- `formatDateTime(date, format?)` — formats any `dayjs`-compatible input, defaulting to `DateFormats.ISO_DATETIME`

**Shared type helpers**: `src/utils/types.utils.ts` holds generic utility types used across the app (`Nullable<T>`, `ValueOf<T>`, `ExtendedString<T>`, `isNullable`).

**API layer**: `src/services/api/` — `api.config.ts` exports the shared axios instance (`requestInstance`) with base headers; `api.service.ts` scaffolds an `ApiService` class that concrete services are expected to extend/use.

**Entry point**: `index.html` loads `src/main.tsx`, which mounts `<App />` (from `components/App`) into `#root` and imports `src/index.scss` for global styles.

**TypeScript config**: project uses TS project references — `tsconfig.json` references `tsconfig.app.json` (covers `src`) and `tsconfig.node.json` (covers `vite.config.ts`). Both enable `verbatimModuleSyntax` (use `import type { ... }` for type-only imports), `noUnusedLocals`/`noUnusedParameters`, and `erasableSyntaxOnly`.

---
> Source: [devsendjin/react-ts-template](https://github.com/devsendjin/react-ts-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
