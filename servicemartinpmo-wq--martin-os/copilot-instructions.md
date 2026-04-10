## martin-os

> Martin OS is a Next.js 14 (App Router) single-process application — a tri-native dashboard combining **PMO-Ops** (host), **Tech-Ops**, and **Miidle** as native plugins. All API routes are colocated; there is no separate backend.

# AGENTS.md

## Cursor Cloud specific instructions

### Overview

Martin OS is a Next.js 14 (App Router) single-process application — a tri-native dashboard combining **PMO-Ops** (host), **Tech-Ops**, and **Miidle** as native plugins. All API routes are colocated; there is no separate backend.

### Running the app

See `README.md` for standard commands (`pnpm dev`, `pnpm build`, `pnpm lint`). `npm run dev` binds to `0.0.0.0:3000` so **localhost** and **127.0.0.1** both work; keep the process running while developing.

### Key caveats

- **Both lockfiles exist** (`pnpm-lock.yaml` and `package-lock.json`). Use **pnpm** as the primary package manager; the `.npmrc` sets `legacy-peer-deps=true`.
- **No external services required for local dev.** Supabase, Pexels API, and Vercel AI Gateway all have graceful fallbacks to demo/static data when env vars are absent.
- **VPS / production:** `npm run start` binds to `127.0.0.1` only. On a server use `npm run vps:deploy` or `npm run start:host`, or `npm run docker:up` + `docker-compose.yml`. Health check: `GET /api/health`. Optional systemd: `scripts/vps-install-service.sh`.
- **Tailwind v4** is used with `@tailwindcss/postcss` (not the older `tailwindcss` PostCSS plugin). Theme CSS lives in `src/styles/`.
- **Path alias:** `@/*` maps to `src/*` (configured in `jsconfig.json`).
- **ESLint 10** with flat config (`eslint.config.js`). The `no-unused-vars` rule ignores variables starting with an uppercase letter or underscore.
- **No automated test suite** exists in this repo (no Jest, Vitest, Cypress, Playwright, etc.). Lint (`pnpm lint`) is the primary automated check.
- **Zustand store** at `src/store/useMartinStore.js` manages global UI state (domain, mode, presence, command center, signals). Access via `useMartinStore()` or `useMartinStore.getState()` for non-React contexts.
- **V2 Theme CSS** is wired in `src/index.css`. Four V2 presets (`enterprise-light`, `cyber-hud`, `dark-glass`, `wellness-soft`) are defined in `src/styles/theme-engine-v2.css` and validated via `src/lib/themePresets.js`.
- **Framework Gallery** lives at `/pmo-ops/frameworks` (Next.js page at `app/pmo-ops/frameworks/page.jsx`). Framework metadata is in `src/registry/frameworkRegistry.js`.
- **Command Center** (⌘K) and **Signal Feed** are globally mounted in `app/providers.jsx`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/servicemartinpmo-wq)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/servicemartinpmo-wq)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
