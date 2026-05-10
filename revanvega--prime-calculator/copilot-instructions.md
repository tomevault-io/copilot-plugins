## prime-calculator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ALIGN Calculator is a retirement income planning tool that helps users compare their current retirement path against an "ALIGN" (Annual Lifetime Income Goals Navigator) path that includes annuity conversions. The app projects income from multiple sources over time, calculates guaranteed income percentages, and runs Monte Carlo simulations.

## Commands

```bash
npm run dev      # Start development server (Next.js)
npm run build    # Production build
npm run lint     # ESLint
```

## Architecture

### State Management
All calculator state lives in `context/CalculatorContext.tsx` which provides:
- An 8-step wizard flow (StepId 1-8)
- Client/spouse information, income sources, accounts, and ALIGN annuity options
- Update functions for each state slice (setClient, setIncome, addAccount, etc.)
- `loadSampleData()` for dev testing with pre-filled data

### Projection Engine
Core financial calculations are in `lib/`:
- **`projection.ts`** - `runProjection(state, usePrimeAnnuity)` generates year-by-year income rows from current age to plan end. Handles COLA adjustments, account growth/distributions, and ALIGN annuity payouts.
- **`monte-carlo.ts`** - `runMonteCarlo()` runs 1000 simulations using historical S&P returns to calculate portfolio success rates
- **`types.ts`** - All TypeScript interfaces (ClientInfo, AccountBucket, AnnuityPrimeInputs, etc.) and default values

### Data Flow
1. User inputs in step components update `CalculatorContext` state
2. `useProjection` hook (in `hooks/useProjection.ts`) computes both "current" and "prime" projections
3. Summary/comparison components consume projections via the hook

### Key Components
- **Steps 1-4**: Data entry (client info, income, guaranteed income, accounts)
- **Step 5**: `CurrentIncomeSummary` with `variant="current"` - current path projection table
- **Step 6**: `AnnuityPrimeSection` - ALIGN annuity configuration
- **Step 7**: `CurrentIncomeSummary` with `variant="prime"` - ALIGN path projection table
- **Step 8**: `ComparisonGraph` - side-by-side comparison with Recharts visualization and Monte Carlo analysis

### Styling
- Tailwind CSS with custom color palette in `tailwind.config.ts`
- CSS custom properties in `globals.css` for column category colors (--col-age, --col-earned, etc.)
- Dark theme by default (bg-black, text-gray-100)

## Key Types

```typescript
AccountType = "qualified" | "roth" | "taxable" | "cash" | "insurance"
IncomeOwner = "client" | "spouse"
PrimeBenefitOption = "singleLife" | "joint"
PrimeProductType = "FIA" | "MYGA"   // FIA = fixed payout; MYGA = fixed % of principal for term (3/5/7 yr)
```

Projections produce `ProjectionRow[]` with annual/monthly totals, guaranteed income amounts/percentages, and account draws by type and owner.

---
> Source: [RevanVega/PRIME-Calculator](https://github.com/RevanVega/PRIME-Calculator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
