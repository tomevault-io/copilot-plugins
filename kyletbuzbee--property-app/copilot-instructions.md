## property-app

> The **Property App** is an institutional-grade real estate triage and investment analysis dashboard specifically designed for fix-and-flip investors in the East Texas market (Tyler, Longview, Marshall, etc.). It automates property evaluation using AI deal scoring, market intelligence (ZHVI/ZORI), and institutional buy parameters.

# Property App: Real Estate Triage Dashboard - GEMINI.md

## Project Overview

The **Property App** is an institutional-grade real estate triage and investment analysis dashboard specifically designed for fix-and-flip investors in the East Texas market (Tyler, Longview, Marshall, etc.). It automates property evaluation using AI deal scoring, market intelligence (ZHVI/ZORI), and institutional buy parameters.

### Core Technology Stack

- **Framework:** Next.js 15+ (App Router)
- **Language:** TypeScript (Strict Mode)
- **Frontend:** React 19, Tailwind CSS, TanStack Table (Data Tables), Recharts (Charts), Leaflet (Maps)
- **Backend:** Next.js API Routes, Prisma ORM
- **Database:** PostgreSQL (Supabase)
- **Testing:** Playwright (E2E)
- **AI Integration:** Custom "East Texas Deal Inspector" persona using RAG (Retrieval-Augmented Generation) with local market data bundles.

---

## Building and Running

### Development Commands

- `npm run dev`: Starts the Next.js development server at `localhost:3000`.
- `npm run build`: Creates a production build.
- `npm run start`: Starts the production server.
- `npm run lint`: Runs ESLint and Stylelint checks.
- `npm run format`: Formats code using Prettier.

### Database Management

- `npx prisma generate`: Generates the Prisma client.
- `npx prisma db push`: Synchronizes the database schema with the Prisma schema.
- `npx prisma studio`: Opens a GUI to view and edit database data.

### Testing

- `npx playwright test`: Runs all E2E tests.
- `npx playwright test --ui`: Opens the Playwright UI for interactive test development.

---

## Architecture & Data Model

### Key Entities (Prisma Schema)

- **Property:** Central entity for deal analysis. Includes specs, AI-derived scores (MAO, deal score), and status.
- **Organization/User:** Multi-tenancy support for investment teams.
- **RehabItem:** Detailed line-item tracking for renovation budgets.
- **MarketData:** Neighborhood-level trends and inventory data.
- **Accounting:** `BankAccount`, `Transaction`, and `Vendor` models for financial tracking.
- **TimelineEvent:** Project management milestones for rehab/listing.

### AI Scoring & Logic (`src/lib/ai/enhancedScoring.ts`, `src/lib/calculations.ts`)

The system enforces strict institutional rules via the "East Texas Deal Inspector" persona:

1. **Rule 1 (Flipping Only):** Ignores rental metrics; focuses solely on retail flip potential.
2. **Rule 2 (Visual Evidence):** AI analyzes images for structural/mechanical distress to adjust rehab tiers.
3. **Rule 3 (Knowledge Base Priority):** Mandatory use of exact local comps, velocity, and rehab catalogs from `KnowledgeBundle`.
4. **Rule 5 (Missing Data Fallback):** Throws `IncompleteDataError` if `listPrice`, `sqft`, or `zip` are missing.
5. **Rule 6 (Output Format):** AI responses follow a strict structure (Narrative + JSON block).

#### Preflight Gates

- **HARD_FAIL:** If List Price > 75% of ARV.
- **HARD_FAIL:** If Market Median DOM > 90 days.
- **CAUTION:** If Market P75 DOM > 90 days.

#### Institutional Formulas

- **Holding Costs:** `($350 * months) + ((ARV * 0.0235) / 12 * months)`. Months = 4 (standard) or 5 (if DOM > 60).
- **MAO (Maximum Allowable Offer):** `ARV - (Rehab * 1.2) - Holding Costs - (ARV * 0.08) - Profit Target`.

---

## Development Conventions

### Coding Standards

- **Naming:** PascalCase for Components/Types, camelCase for Variables/Functions, snake_case for Database columns.
- **Imports:** Use the `@/` alias for `src/` directory. Order: React/Next -> Third-party -> Internal -> Types.
- **State:** Prefer React Context for global UI state and TanStack Table for complex data filtering/sorting.

### API Patterns

- **Response Format:** All API routes return `{ success: boolean; data?: any; error?: string }`.
- **Data Conversion:** DB uses snake_case; Client uses camelCase. Use `serializeProperty` and `addCalculations` helpers in `src/lib/calculations.ts`.

### Directory Structure

- `src/app/`: Next.js App Router (Pages & API Routes).
- `src/components/`: Modular React components (Dashboard, Maps, Tables).
- `src/lib/`: Core logic (AI scoring, market data loaders, calculations).
- `src/data/`: Static constants, types, and initial local data.
- `processed/`: Curated CSV/JSON market data (e.g., `gem_knowledge_bundle.csv`) used by the RAG engine.

---

## Environment Configuration

Ensure `.env.local` contains the following:

```bash
DATABASE_URL="your-postgresql-url"
NEXT_PUBLIC_SUPABASE_URL="your-supabase-url"
NEXT_PUBLIC_SUPABASE_ANON_KEY="your-anon-key"
```

Refer to `AGENTS.md` for detailed coding standards and `P4C_PROJECT_INTELLIGENCE.md` for project history and audit findings.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kyletbuzbee)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kyletbuzbee)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
