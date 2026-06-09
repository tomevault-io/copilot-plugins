## deni

> - `npm run dev` — dev server

# Deni — Claude guidance

## Commands
- `npm run dev` — dev server
- `npm run build` — production build (runs tsc first)
- `npm run test` — Vitest unit tests (42 tests, all in `src/lib/finance.test.ts`)
- `npm run lint` — ESLint strict

## Critical rules
1. **Financial logic only in `src/lib/finance.ts`**. No money math in components.
2. **No backend calls, ever.** Local-only (Dexie IndexedDB). Do not add fetch/axios/supabase.
3. **No new dependencies without asking.** Stack is fixed: React 18, Vite, TS strict, Tailwind, react-router, Dexie, vite-plugin-pwa, Vitest.
4. **TypeScript strict — no `any`.** Check with `npx tsc -p tsconfig.app.json --noEmit`.
5. **Currency always via `fmtKES()`** from `src/lib/finance.ts`. Never raw `toFixed`.
6. **Lender rates in `src/data/lenders.ts` only.** Update the `// Last reviewed:` comment when changing rates.

## Architecture
```
src/
  lib/finance.ts        — pure financial functions (APR, severity, avalanche, early-repay)
  lib/finance.test.ts   — 42 Vitest tests
  data/lenders.ts       — lender presets + disclaimer copy
  store/db.ts           — Dexie schema (debts, payments, settings)
  screens/              — PlanScreen, CalculatorScreen, PaymentsScreen, LearnScreen
  App.tsx               — BrowserRouter + bottom nav
```

---
> Source: [chepkemboiedel-sudo/Deni](https://github.com/chepkemboiedel-sudo/Deni) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
