## budget-vue

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development
npm run dev          # Start Vite dev server (proxies /api → localhost:8080)
npm run build        # Production build to dist/
npm run preview      # Preview production build

# Quality
npm run lint         # Type check + ESLint
npm run format       # Prettier formatting
npm run type-check   # TypeScript check only

# Tests
npm run test         # Watch mode
npm run test:run     # Single run
npm run test:ui      # Vitest UI dashboard
```

To run a single test file: `npx vitest run src/__tests__/transactions.test.ts`

## Architecture

**Stack:** Vue 3 (Composition API + `<script setup>`) · TypeScript · Vite · Vuetify 3 · Pinia · Vue Router 4 · Axios · Apache ECharts

**Structure:**
- `src/pages/` — Route-level components (one per route)
- `src/components/` — Reusable UI components (`charts/`, `forms/`, `shared/`)
- `src/stores/` — Pinia stores (state + methods, Composition API style)
- `src/composables/` — Shared logic (`useApi`, `useSnackbar`, `useMoneyFormat`)
- `src/router/` — Route definitions + navigation guard
- `src/__tests__/` — Vitest unit tests with setup mocks

### Authentication flow

A single API key is stored in `sessionStorage` via `stores/session.ts`. The `useApi()` composable creates a singleton Axios instance that injects the key via request interceptor and handles 401s by clearing the session and redirecting to `/login`.

### Store pattern

Stores use `defineStore()` with Composition API style and explicit return objects. Persistent preferences (theme, drawer state) use `localStorage`; the API key uses `sessionStorage`. Each store exposes `loadFromStorage()` called during app startup in `main.ts`.

### Money values

All monetary values are integers in **pence** (GBP). Use `useMoneyFormat()` to render them as `£X.XX` strings. Never store or compute with decimal amounts.

### Responsive layout

Mobile breakpoint is `windowWidth < 600`. Mobile uses a temporary navigation drawer; desktop shows top-bar nav buttons. Detection is done in `App.vue` via a resize listener.

### Testing conventions

- Tests live in `src/__tests__/` as `*.test.ts`
- `src/__tests__/setup.ts` mocks Vuetify components, ECharts, `matchMedia`, and `ResizeObserver`
- Use `setActivePinia(createPinia())` in `beforeEach` for store tests
- API calls are mocked with `vi.mock()`
- Files with `.test.ts.disabled` extension are intentionally skipped

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/piotrzalecki) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
