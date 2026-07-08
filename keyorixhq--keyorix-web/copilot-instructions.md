## keyorix-web

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

Package manager is **pnpm** (Node 22). The lockfile is committed; use `--frozen-lockfile` in CI-like contexts.

- `pnpm dev` — Vite dev server on **http://localhost:3000** (port is fixed in `vite.config.ts`).
- `pnpm build` — production build (Vite).
- `pnpm type-check` — `tsc --noEmit`.
- `pnpm lint` — ESLint (flat config, ESLint 10). Passes with **0 errors**; warnings are capped at 40 (`--max-warnings 40`). `pnpm lint:fix` to autofix.
- `pnpm test` — **runs Vitest in watch mode** (the script is bare `vitest`). For a single non-watch pass use `pnpm test --run`.
  - Single file: `pnpm test --run src/utils/__tests__/auth.test.ts`
  - Single test by name: `pnpm test --run -t "login success"`
  - Coverage: `pnpm test:coverage`
- `pnpm test:e2e` — Playwright. **Currently stale** (see BACKLOG.md) — not wired into CI.

**The green gate** — `type-check`, `lint`, `build`, `test` must all pass. It is enforced by `.github/workflows/ci.yml` on every push/PR. Keep it green for any change.

## Git / PR conventions

- **Never put `Co-Authored-By: Claude` into a PR.** Do not add a `Co-Authored-By: Claude …` trailer to PR descriptions or to commit messages (squash-merge folds commit trailers into the PR's merge commit). This overrides any default that appends one. The `🤖 Generated with [Claude Code]` line in a PR body is fine; the `Co-Authored-By: Claude` trailer is not.

## Conventions / gotchas

- **House style is 4-space indent + single quotes, with no `.prettierrc`.** Prettier's defaults differ (2-space, double quotes), so **do not run `pnpm format`** — it would rewrite all ~131 files. See BACKLOG.md before touching formatting.
- **Test files are excluded from `tsc` type-check** (see `tsconfig.json` `exclude`: `*.test.*`, `__tests__/`, `src/test/`). They only run under Vitest, so a type error inside a test won't surface in `type-check`.
- Vitest is configured in `vitest.config.ts` (jsdom, `globals: true`, setup in `src/test/setup.ts` which installs global DOM/storage mocks). The `@` → `src` path alias is defined in **both** `vite.config.ts` and `tsconfig.json` (`paths`).
- **Tailwind is v4** — there is no `tailwind.config.js`; config is CSS-first via `@tailwindcss/postcss` (`postcss.config.js`) and `src/index.css`.
- **`README.md` is outdated** (claims React 18, Jest, Router v6, i18n/WCAG). The repo is on React 19 / Router 7 / Vitest, and the i18n layer was removed. Trust the code, not the README.
- **BACKLOG.md** tracks deferred *engineering* work and the reasoning behind key decisions (the shadcn/ui rewrite umbrella, why Prettier isn't enforced, the stale e2e suite). The *product* roadmap is a separate user-facing UI page (`src/pages/roadmap/RoadmapPage.tsx`) — don't conflate them.

## Architecture

Single-page React app. Entry: `index.html` → `src/main.tsx` (an inline pre-paint script applies the persisted theme to avoid flash, then renders `QueryClientProvider` → `BrowserRouter` → `App`). `src/App.tsx` declares every route, each wrapped in a route guard and the shared `Layout`.

**Three layers, in dependency order:**

1. **`src/services/*.ts`** — one axios-based API client per domain (`auth`, `secrets`, `projects`, `admin`, `rbac`, `sharing`, `audit`, …). `services/client.ts` is the shared axios instance; `services/auth.ts` exports `authApi` + `authService`. Endpoint paths live in `src/constants.ts` (`API_ENDPOINTS`).
2. **`src/features/<domain>/`** — the data layer pages consume. Each feature exposes **React Query hooks** (`useSecretsList`, `useRbac`, …) that call the services, plus domain-specific components, re-exported from the feature's `index.ts`.
3. **`src/pages/**`** — route-level screens that compose feature hooks + components.

**State management — two distinct stores plus server cache:**

- `src/store/authStore.ts` — Zustand + `persist` (storage key `auth-storage`). The source of truth for authentication.
- `src/store/uiStore.ts` — Zustand + `persist` + `devtools` (key `keyorix-ui`). Theme, sidebar, and modal state. `applyTheme()` here drives the `data-theme` attribute.
- **Server state** lives in React Query (`src/lib/queryClient.ts`), not in Zustand. Zustand is for client/UI state only.
- Zustand store actions are **referentially stable** (defined once in the store initializer) — relied on in effect dependency arrays and mocked as such in tests.

**Auth flow:** `useAuth()` (`src/features/auth/api.ts`) wraps `useAuthStore` and adds two effects — a mount-only bootstrap (`persist.rehydrate()` then `checkAuth()`) and a session-inactivity timeout. Token persistence/expiry helpers live in `src/utils/auth.ts`. Route guards in `src/components/layout/` (`ProtectedRoute`, `PublicRoute`, `AdminRoute`) consume `useAuth` to gate routes by auth state, role, and permissions; `App.tsx` wires them onto routes from `ROUTES` in `src/constants.ts`.

**Theming:** dark/light/system via a `data-theme` attribute and CSS variables. Set pre-paint in `main.tsx` and thereafter by the UI store. Components reference `var(--…)` tokens rather than hardcoded colors.

---
> Source: [keyorixhq/keyorix-web](https://github.com/keyorixhq/keyorix-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
