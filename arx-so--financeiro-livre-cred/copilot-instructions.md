## financeiro-livre-cred

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm run dev` — Start dev server (port 8080)
- `npm run build` — Production build (Vite + TypeScript)
- `npm run lint` — ESLint check (flat config, Airbnb-based)
- `npm run preview` — Preview production build
- No test framework is configured

## Architecture

**LivreCred** — Financial management system (Portuguese/Brazilian). React 18 + TypeScript + Vite, backed by Supabase.

### Data flow

```
Pages (src/pages/) → Hooks (src/hooks/) → Services (src/services/) → Supabase
```

- **Services** make Supabase queries and return typed data
- **Hooks** wrap services with TanStack React Query (`useQuery`/`useMutation`) and manage cache keys
- **Pages** consume hooks, wrapped in `AppLayout`
- **Stores** (Zustand): `useAuthStore` (auth state), `useBranchStore` (selected branch, persisted)

### Types

`src/types/database.ts` mirrors Supabase schema with `Row`/`Insert`/`Update` per table. Key convenience types: `BudgetCategoryWithSubcategories`, `BudgetSubcategoryData`, `BudgetMonthData`.

### Key shared components

`StatCard`, `PageHeader`, `LoadingState`, `EmptyState`, `CurrencyInput`, `SearchInput`, `AdvancedFilters` — all in `src/components/shared/`.

### Routing

React Router v6 in `src/App.tsx`. `ProtectedRoute` wraps authenticated pages, `PublicRoute` wraps login. Providers: QueryClientProvider → ThemeProvider → TooltipProvider → BrowserRouter → AuthInitializer.

### Planejamento (Budget) module

Most complex module. `src/pages/Planejamento/` has 9 component files for hierarchical budget display. Budget versions managed by `src/services/budgetVersions.ts`. `budget_items` table uses upserts with conflict keys that vary by whether subcategory_id is present.

## Code Style

- **4-space indentation** (enforced by ESLint)
- Path alias: `@/*` → `src/*`
- Import order: external libraries first, then `sonner`, then local `@/` imports
- ESLint enforces `implicit-arrow-linebreak`, `react/jsx-closing-bracket-location`
- `jsx-a11y/label-has-associated-control` warnings are pre-existing and accepted
- Max line length: 120 chars (warning)
- `src/components/ui/` is **auto-generated Shadcn** — do not modify these files
- `supabase/` directory is ignored by ESLint

## Environment

Requires `.env` with `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY`. See `.env.example`.

## Domain vocabulary

- **Unidade/Branch** — company branch/location (multi-tenant by branch)
- **Favorecido** — client, supplier, or employee entity
- **Receita/Despesa** — income/expense
- **Folha de Pagamento** — payroll
- **Conciliação** — bank reconciliation
- **Orçamento** — budget
- **Metas** — sales targets
- **Comissões** — sales commissions
- **DRE** — income statement report

---
> Source: [Arx-so/financeiro-livre-cred](https://github.com/Arx-so/financeiro-livre-cred) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
