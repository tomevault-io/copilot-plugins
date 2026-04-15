## takehome-pay-calculator

> Take-Home Pay Calculator — a Next.js web app for South African contractors to calculate net take-home pay after taxes (SARS 2025/26 brackets), deductions, and expenses. Supports ZAR and USD currencies.

# CLAUDE.md

## Project Overview

Take-Home Pay Calculator — a Next.js web app for South African contractors to calculate net take-home pay after taxes (SARS 2025/26 brackets), deductions, and expenses. Supports ZAR and USD currencies.

## Tech Stack

- **Framework:** Next.js 16, React 19, TypeScript 5
- **Styling:** Tailwind CSS 4 (CSS-first config, dark mode via custom variant), shadcn/ui (Radix UI primitives)
- **State:** Zustand (single store in `lib/store.ts` with all calculation logic)
- **Charts:** Recharts
- **Forms:** React Hook Form + Zod validation
- **Export:** xlsx (Excel), pdf-lib (PDF)
- **Package manager:** pnpm (preferred), npm also works

## Commands

```bash
npm run dev       # Start dev server (Turbopack)
npm run build     # Production build (Turbopack)
npm run lint      # Run ESLint (next lint)
npm run start     # Run production server
npm test          # Run tests (vitest)
```

## Project Structure

```
app/                  # Next.js App Router
  layout.tsx          # Root layout (metadata, viewport, theme provider)
  page.tsx            # Home page (renders Calculator component)
  globals.css         # Global styles + Tailwind v4 theme (CSS-first config)
components/           # Feature components
  calculator.tsx      # Main 5-tab calculator interface
  input-form.tsx      # Income & deduction inputs
  dashboard.tsx       # Charts and metrics display
  other-expenses.tsx  # Dynamic expense tracking
  depreciation.tsx    # Wear-and-tear depreciation
  export-options.tsx  # Excel & PDF export
  ai-chat.tsx         # AI assistant chatbot
  clear-all-button.tsx
  theme-provider.tsx
  ui/                 # shadcn/ui components (~50 files, auto-generated)
hooks/                # Custom hooks (use-mobile.tsx, use-toast.ts)
lib/
  store.ts            # Zustand store — ALL state + tax calculation logic
  utils.ts            # cn(), formatCurrency(), formatPercentage()
__tests__/            # Vitest test suites
  calculator.test.ts       # Tax bracket & deduction calculations
  clear-all.test.ts        # Reset functionality
  numeric-input.test.tsx   # Input validation (component)
  store-expenses.test.ts   # Expenses, depreciation, office fraction
  currency-conversion.test.ts  # USD/ZAR conversion & integration
  dashboard.test.tsx       # Dashboard rendering & metrics
public/               # Static assets, PWA manifest, service worker
```

## Key Architecture Decisions

- **All calculation logic lives in `lib/store.ts`** — tax brackets, medical credits, deductions, and derived values are computed in the Zustand store's `calculateResults()` method. Do not duplicate this logic elsewhere.
- **Components use `"use client"` directive** — this is a client-side interactive app.
- **Path alias:** `@/*` maps to the project root (configured in tsconfig.json).
- **Build config:** Strict builds (TypeScript and ESLint errors must be fixed). Optimized for Vercel deployment.
- **shadcn/ui components in `components/ui/`** are auto-generated — avoid hand-editing these files.

## Code Conventions

- **File naming:** lowercase with hyphens (`input-form.tsx`, `clear-all-button.tsx`)
- **Component naming:** PascalCase (`Calculator`, `InputForm`)
- **Hooks:** prefixed with `use-` (`use-mobile.tsx`, `use-toast.ts`)
- **Variables:** camelCase, financial terms kept descriptive (`grossIncomeZAR`, `monthlyPAYE`, `taxableAnnual`)
- **Currency:** ZAR is the base currency; USD converted at a fixed rate (1 USD = 18.5 ZAR)

## Testing

Tests use Vitest with jsdom. Run with `npm test`. Test files live in `__tests__/` and import directly from `lib/store.ts`. When modifying tax calculations, update corresponding tests in `calculator.test.ts`.

## Common Tasks

- **Adding a new deduction:** Add state field + setter to `CalculatorState` interface in `lib/store.ts`, include in `calculateResults()`, add UI input in `components/input-form.tsx`.
- **Updating tax brackets:** Modify the bracket arrays in the `calculateResults()` function in `lib/store.ts`.
- **Adding a new UI component:** Use `npx shadcn@latest add <component>` for shadcn/ui primitives, or create feature components in `components/`.
- **Tailwind theme:** All theme colors and design tokens are defined in `app/globals.css` using `@theme inline` (no `tailwind.config.ts`).

## Workflow

- **Large refactors:** Break into phases — research, plan, implement, validate, document. Use the `/refactor-plan` skill for guided refactoring.
- **Before committing:** Always run `npm test` and `npm run build` to catch issues.
- **Session start hook:** Automatically runs tests and build check at the start of each Claude Code session (configured in `.claude/settings.json`).
- **Dependencies:** Always pin to specific version ranges (`^x.y.z`), never use `"latest"`.
- **Strict builds:** TypeScript and ESLint errors must be fixed — do not use `ignoreBuildErrors` or `ignoreDuringBuilds`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jslabxyz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jslabxyz)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
