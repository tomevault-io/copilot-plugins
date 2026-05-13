## rent-or-buy-homes

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A production-ready buy vs rent calculator that helps users compare long-term net-worth outcomes between buying a home and renting while investing the savings. The app is 100% client-side (no backend), deployed on Vercel at [rent-or-buy.homes](https://rent-or-buy.homes).

## Current Implementation Status

**The application is feature-complete and production-ready.** All major features are implemented:

- ✅ **Complete UI/UX**: Full responsive design with all input panels, charts, and visualizations
- ✅ **Financial Engine**: Sophisticated calculations with mortgage amortization, tax optimization, and investment projections
- ✅ **Internationalization**: Full English/Chinese support via i18next with browser language detection
- ✅ **URL State Sync**: Shareable links with LZ-compressed state parameters
- ✅ **Charts & Visualizations**: ApexCharts integration for net worth and cash outflow projections
- ✅ **City Presets**: Pre-configured market data for major US cities
- ✅ **Analytics**: Vercel Analytics integration

### Future Enhancements

- [ ] PDF export functionality for detailed reports
- [ ] Additional chart types for advanced financial projections
- [ ] More city presets and regional data

## Development Commands

- `npm run dev` - Start development server with hot reload
- `npm run build` - Build for production (TypeScript compilation + Vite build)
- `npm run lint` - Run ESLint to check code quality
- `npm run preview` - Preview production build locally

## Tech Stack & Architecture

### Core Technologies

- **React 19** with **TypeScript** - Component framework with strict typing
- **Vite** - Build tool and development server
- **Tailwind CSS** - Utility-first styling framework
- **ApexCharts** - Interactive charting library
- **React Router v7** - Client-side routing
- **i18next** - Internationalization framework with browser language detection
- **LZ-String** - URL compression for shareable state

### Architecture Overview

```
┌──────────────┐     state updates      ┌──────────────┐
│  InputPanel  │ ────────────────────►  │   AppContext │
│              │                        │  (useReducer)│
└──────────────┘                        └───────┬──────┘
                                                │
                                                ▼
┌──────────────┐                        ┌──────────────┐
│   URLSync    │◄─── history.replace ───│ HeroSection  │
│   Hook       │                        │ ResultPanel  │
└──────────────┘                        └──────────────┘
```

**Key Architectural Decisions:**

- **State Management**: Centralized via React Context with `useReducer` for predictable updates
- **Calculations**: Pure TypeScript functions in `src/lib/finance/` - no side effects, easy to test
- **URL Sync**: Automatic bidirectional sync between app state and URL parameters using `useURLSync` hook
- **i18n Integration**: Language state managed in AppContext, synced with i18next on language changes

## Project Structure

```
src/
├── components/
│   ├── Header/               - Language selector and branding
│   ├── HeroSection/          - Time horizon selector, conclusion badge, net worth cards
│   ├── InputPanel/           - Buy/Rent scenario inputs with shared UI components
│   │   ├── shared/           - SliderInput, NumberInput, ButtonGroup
│   │   ├── BuyInputs.tsx     - Property purchase parameters
│   │   └── RentInputs.tsx    - Rental and investment parameters
│   ├── ResultPanel/          - Charts and visualizations
│   │   ├── NetWorthChart.tsx - ApexCharts line chart for net worth projection
│   │   └── CashOutflowChart.tsx - ApexCharts bar chart for cash flow analysis
│   ├── ShareButton/          - URL generation for sharing calculator state
│   ├── DebugPanel/           - Development tool (toggled via URL ?debug=1)
│   └── Footer/               - Legal disclaimers and links
├── lib/
│   ├── finance/              - Core calculation engine
│   │   ├── calculations.ts   - Pure functions for mortgage, tax, investment math
│   │   └── types.ts          - TypeScript interfaces for financial data
│   ├── constants.ts          - Investment presets, tax brackets, defaults
│   ├── inputUtils.ts         - Input validation and formatting utilities
│   ├── urlSync.ts            - LZ-string compression/decompression for URL state
│   └── design-system.ts      - Tailwind CSS utility classes and theme constants
├── contexts/
│   └── AppContext.tsx        - Global state management with useReducer
├── data/
│   └── cityDefaults.ts       - Market-specific presets (Seattle, SF, LA, etc.)
├── hooks/
│   ├── useCalculations.ts    - Hook that orchestrates financial calculations
│   ├── useURLSync.ts         - Bidirectional URL ↔ state synchronization
│   └── useDebugMode.ts       - Debug panel toggle logic
├── locales/                  - i18next translation files (en/zh)
├── App.tsx                   - Main layout and component orchestration
└── i18n.ts                   - i18next configuration
```

## Financial Calculation Engine

The core calculation logic is in `src/lib/finance/calculations.ts`. All functions are pure (no side effects) for testability.

### Key Calculations

**Mortgage Amortization** (`calculateMonthlyPayment`, `buildAmortizationSchedule`)
- Standard PMT formula for monthly payment calculation
- Full amortization schedule with principal/interest breakdown per payment
- Handles scenarios where loan is paid off before projection horizon

**Property Appreciation** (`calculatePropertyAppreciation`)
- Compound annual growth: `FV = PV × (1 + r)^n`
- Customizable appreciation rates

**Investment Growth** (`calculateInvestmentReturns`)
- Portfolio growth with annual contributions (differential cash flow between scenarios)
- Compound returns with reinvestment

**Tax Calculations**
- Mortgage interest deduction (itemized vs. standard deduction comparison)
- Property capital gains with $250k/$500k exemptions (single/married filing)
- Investment capital gains (long-term vs. short-term rates)
- Different marginal tax rates by filing status

**Cash Flow & Net Worth**
- Year-by-year cash outflow tracking for both scenarios
- Final net worth calculation includes:
  - Property value minus remaining mortgage balance and selling costs
  - Investment portfolio value minus capital gains tax
- "Same money in" principle: when one scenario costs less in a year, the difference is invested

### Calculation Reference

An Excel version of the calculator is available at `reference/validate.xlsx` for manual validation of calculation logic.

## Important Implementation Details

### State Management Pattern

The app uses a centralized state management pattern via `AppContext`:

```typescript
// State is managed via useReducer with action-based updates
const { state, dispatch } = useApp();

// Update patterns:
updateBuyInput(field, value)    // Buy scenario inputs
updateRentInput(field, value)   // Rent scenario inputs
updateAppSetting(field, value)  // App-level settings (time horizon, language)
```

All calculations are derived from state in `useCalculations` hook, not stored in state themselves. This ensures calculations always reflect current inputs.

### URL Synchronization Pattern

`useURLSync` hook provides bidirectional sync:
- **State → URL**: Whenever state changes, URL is updated via `history.replaceState`
- **URL → State**: On mount, URL params are parsed and applied to state
- Compression via `lz-string` keeps URLs shareable despite large state objects

### Internationalization Pattern

- Translation files in `src/locales/en/` and `src/locales/zh/`
- Language state stored in `AppContext.appSettings.currentLanguage`
- Browser language detection via `i18next-browser-languagedetector`
- Sync i18n with app state on language changes in `App.tsx`

### Debug Mode

Access debug panel by adding `?debug=1` to URL. Shows:
- Full app state (inputs, calculations, settings)
- Useful for development and troubleshooting user-reported issues

### City Presets

`src/data/cityDefaults.ts` contains market-specific defaults:
- Home prices, rent amounts, property tax rates
- Data sourced from CRMLS and other regional MLS systems
- Applied via city selector in HeroSection

## Key Files to Understand

1. **`src/contexts/AppContext.tsx`** - Global state shape and reducer logic
2. **`src/lib/finance/calculations.ts`** - All financial calculation formulas
3. **`src/hooks/useCalculations.ts`** - Orchestrates calculations from state
4. **`src/lib/urlSync.ts`** - URL compression/decompression logic
5. **`src/data/cityDefaults.ts`** - City-specific market data

---
> Source: [zebangeth/rent-or-buy.homes](https://github.com/zebangeth/rent-or-buy.homes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
