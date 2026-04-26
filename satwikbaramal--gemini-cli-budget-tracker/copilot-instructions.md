## gemini-cli-budget-tracker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development
npm run dev              # Start dev server (0.0.0.0:3000)
npm run build            # Production build
npm run lint             # ESLint

# Testing
npm test                 # Run all tests (Jest + jsdom)
npm run test:watch       # Watch mode
npm run test:coverage    # Coverage report

# Database utilities
npm run populate-data    # Seed test data
npm run clear-user-data  # Clear all user data
npm run re-encrypt       # Re-encrypt existing data with new key
```

## Architecture

**Vivaranam** — a budget tracking app with AI insights, savings goals, and dual yearly/monthly tracking.

**Stack**: Next.js 15 (App Router, React 19), TypeScript (strict), Tailwind CSS 4, shadcn/ui (Radix), MongoDB/Mongoose, NextAuth v5 (JWT sessions), ExcelJS for exports.

### Encryption

All financial amounts are encrypted at rest with AES-256-GCM (`src/lib/encryption.ts`). The stored format is `iv:authTag:encryptedData` (hex). Always use `decrypt()` when reading and `encrypt()` when writing amounts. Encrypted fields: expense amounts, income values, goal amounts/contributions, fixed expense amounts/overrides.

### Authentication

NextAuth v5 with Credentials (email/password + bcryptjs) and Google OAuth providers. Config split between `src/lib/auth.ts` (full, server-only) and `src/lib/auth.config.ts` (Edge-compatible subset used by middleware). Middleware (`src/middleware.ts`) protects all routes except `/sign-in`, `/sign-up`, and `/api/auth/*`.

### AI Integration (FinBot)

`src/app/api/expenses/summarize/route.ts` — streaming SSE endpoint using OpenAI SDK pointed at GitHub Models (`https://models.github.ai/inference`, model `openai/gpt-4.1-nano`). Requires `GITHUB_TOKEN`. The system prompt includes temporal awareness (past/current/future month handling), full financial context, and security guardrails. Frontend in `src/components/Summary.tsx` renders streamed markdown with typewriter effect.

### Data Models (`src/lib/models/`)

- **Expense** — name, encrypted amount, type (`yearly`/`monthly`), month, date, year. Indexed on (userId, year, type).
- **FixedExpense** — recurring expenses with `applicableMonths` array. **FixedExpenseOverride** — per-month amount overrides.
- **Goal** — savings goals with encrypted target/current amounts, contributions array (each with encrypted amount, type, note).
- **Setting** — key-value store for `yearlyIncome`/`monthlyIncome` (encrypted). **MonthlyIncomeOverride** — per-month income overrides.
- **FilterPreset** — saved search/filter configurations.

All models are scoped by `userId` with compound indexes.

### Key Patterns

- **Path alias**: `@/*` maps to `./src/*`
- **API routes** follow REST-ish conventions under `src/app/api/`. Most use `POST` for create/update/delete with action discrimination in the body.
- **Dual tracking**: Yearly view (`/`) and monthly view (`/monthly`) with separate income and expense management per mode.
- **Fixed expenses** can be overridden per-month without modifying the base record.
- **Theme**: Light/Dark/System via React context (`src/contexts/ThemeContext.tsx`), persisted to both localStorage and database.
- **Export**: `src/app/api/export/excel/route.ts` generates multi-year Excel workbooks with optional AI insight sheets.

### Environment Variables

Required: `MONGODB_URI`, `ENCRYPTION_KEY` (64-char hex), `NEXTAUTH_URL`, `NEXTAUTH_SECRET`
Optional: `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `GITHUB_TOKEN` (for AI features)

Generate encryption key: `node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SatwikBaramal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
