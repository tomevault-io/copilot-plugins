## kimi-portfolio

> This file provides guidance to WARP (warp.dev) when working with code in this repository.

# AGENTS.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Repo layout
- Frontend app lives in `app/` and is built with Vite + React 19 + TypeScript + Tailwind CSS.
- TypeScript path alias: `@/*` → `app/src/*` (see `app/tsconfig*.json` and `app/vite.config.ts`).

## Common commands (run from `app/`)
- Install dependencies (pnpm is implied by `pnpm-lock.yaml`):
  - `pnpm install`
- Start dev server:
  - `pnpm dev` (or `pnpm dev:open` to open browser and bind host)
- Type-check and build production bundle:
  - `pnpm build` (runs `tsc -b` then `vite build`)
- Preview the production build locally:
  - `pnpm preview`
- Lint the codebase with ESLint flat config:
  - `pnpm lint`
- Lint a single file (example):
  - `pnpm exec eslint src/pages/HomePage.tsx`
- Tests: no test runner or scripts are configured in `package.json`.

## Architecture overview
- Entry and bootstrapping
  - Static HTML: `app/index.html` defines the `#root` mount.
  - React mount: `app/src/main.tsx` imports global styles (`index.css`) and renders `<App />`.
- Routing
  - Browser router in `app/src/App.tsx` with routes:
    - `/` → `HomePage` (`app/src/pages/HomePage.tsx`)
    - `/trabalhos` → `WorksPage` (`app/src/pages/WorksPage.tsx`)
  - To add a new route, create a component in `app/src/pages/` and register a `<Route>` in `App.tsx`.
- UI and composition
  - Reusable primitives in `app/src/components/ui/` (Radix-inspired wrappers styled with Tailwind).
  - Feature components in `app/src/components/` (e.g., `ProjectCard`, `TechOrbit`, `FluidBackground`, etc.).
  - Page sections in `app/src/sections/` (`Hero`, `About`, `Projects`, `TechStack`, `Contact`) used by pages to assemble layouts.
- State/logic utilities
  - Custom hooks in `app/src/hooks/` (`use-mobile`, `useMousePosition`, `useScrollVelocity`) encapsulate UI/animation behavior.
  - Shared helpers in `app/src/lib/utils.ts`.
- Content
  - Markdown content lives in `app/src/content/` (e.g., `trabalhos-realizados.md`) and can be rendered with `react-markdown` + `remark-gfm`.
- Styling and theming
  - Tailwind config: `app/tailwind.config.js` (dark mode via `class`, extended design tokens).
  - PostCSS: `app/postcss.config.js`.
  - Global CSS variables, animations, and utilities in `app/src/index.css`.
- Tooling configuration
  - Vite config: `app/vite.config.ts` sets `base: './'`, enables `@` alias, and uses `@vitejs/plugin-react` plus `kimi-plugin-inspect-react`.
  - TypeScript: project references (`app/tsconfig.json` references `tsconfig.app.json` and `tsconfig.node.json`); the build uses incremental TS via `tsc -b`.
  - ESLint: flat config in `app/eslint.config.js` (JS + TS + React Hooks + React Refresh rules).

---
> Source: [looizinho/kimi-portfolio](https://github.com/looizinho/kimi-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
