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

- **Primary Color:** Soft Indigo (`#6366F1`)
- **Background:** Soft Mesh Gradient (Lavender `#F3E8FF` top-left to Warm Cream `#FFF7ED` bottom-right)
- **Surface:** White (`#FFFFFF`) cards with `rounded-xl` and subtle shadows (`shadow-sm`)
- **Typography:** 'Inter' (Sans-serif). Headings bold, labels clear.
- **Layout:**
  - **Header:** Title with status badges ("Beta", "Student") and utility links.
  - **Content:** Two-column grid. Left: Input forms (Loan Parameters). Right: Results summaries and visualizations.
  - **Footer:** Centered disclaimer banner.

---
> Source: [adiagarwalrock/LoanEase](https://github.com/adiagarwalrock/LoanEase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
