## loanease

> LoanEase is a Next.js-based web application designed to calculate loan details. It provides users with tools to input loan parameters (principal, rate, duration), select interest types (simple/compound), and view detailed amortization schedules and payment summaries.

# LoanEase - Project Context

## Project Overview
LoanEase is a Next.js-based web application designed to calculate loan details. It provides users with tools to input loan parameters (principal, rate, duration), select interest types (simple/compound), and view detailed amortization schedules and payment summaries.

## Tech Stack
- **Framework:** Vite + Vue 3 (Composition API)
- **Language:** TypeScript
- **Styling:** Tailwind CSS
- **UI Components:** shadcn-vue (built on Radix Vue primitives), Lucide Vue for icons
- **Validation:** Zod
- **Analytics:** Vercel Analytics, Speed Insights

## Directory Structure
- `src/components/`: Vue components (*.vue files)
  - `ui/`: shadcn-vue UI components (Button, Card, Input, etc.)
  - `LoanCalculator.vue`, `LoanForm.vue`, `AmortizationDisplay.vue`: Domain components
- `src/lib/`: Utility functions, types, and calculation logic
- `src/App.vue`: Root Vue component
- `src/main.ts`: Application entry point
- `docs/`: Project documentation

## Key Features
1.  **Loan Calculator:** Inputs for principal, interest rate, and duration.
2.  **Interest Toggle:** Support for both Simple and Compound interest.
3.  **Amortization Schedule:** Tabular display of payment breakdown over time.
4.  **Visualizations:** Charts reflecting payment schedules (likely using Recharts).

## Development Workflow

### Installation
```bash
npm install
```

### Running Development Server
The development server runs on port 9002.
```bash
npm run dev
```


### Building for Production
```bash
npm run build
```

### Code Quality
- **Type Checking:** `npm run build` (includes vue-tsc)

## Design System Guidelines
- **Primary Color:** Deep Blue (`#3F51B5`)
- **Background:** Light Gray (`#ECEFF1`)
- **Accent:** Lighter Blue (`#5C6BC0`)
- **Typography:** 'Inter' (Sans-serif)
- **Layout:** Clean, tabular layouts for data; geometric icons for concepts. Minimal and uncluttered design.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adiagarwalrock) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
