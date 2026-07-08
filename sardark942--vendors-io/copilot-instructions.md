## vendors-io

> **App:** Chicago Desi Wedding Vendor Marketplace (vendors.io)

# Cursor Rules for Chicago Desi Wedding Vendor Marketplace

## Project Context
**App:** Chicago Desi Wedding Vendor Marketplace (vendors.io)
**Stack:** Next.js 14 (App Router) · Supabase · Stripe Connect · OpenAI · Resend · Tailwind CSS
**Stage:** MVP Development (10-week build, Feb–Apr 2026)
**User Level:** B (Developer with database/API expertise, AI-assisted workflow)

## Directives
1. **Master Plan:** Always read `AGENTS.md` first. It contains the current phase, tasks, and project state.
2. **Documentation:** Refer to `agent_docs/` for detailed context (load only the relevant file):
   - `agent_docs/tech_stack.md` — Stack, setup commands, project structure, env vars
   - `agent_docs/code_patterns.md` — Architecture rules, patterns, naming, anti-patterns
   - `agent_docs/project_brief.md` — Conventions, quality gates, key decisions
   - `agent_docs/product_requirements.md` — Features, user stories, success metrics, scope
   - `agent_docs/testing.md` — Test strategy, commands, pre-commit hooks
3. **Plan-First:** Propose a brief plan and wait for approval before coding. State which files will be created or modified.
4. **Incremental Build:** Build one small feature at a time. Verify after each.
5. **Pre-Commit:** Run lint + typecheck before commits. Fix failures.
6. **No Linting Role:** Do not act as a linter. Use `npm run lint` and `npm run typecheck` to check code.
7. **Communication:** Be concise. Ask ONE clarifying question when needed. No apologies, no filler.

## Commands
- `npm run dev` — Start Next.js dev server
- `npm run build` — Production build
- `npm run lint` — ESLint check
- `npm run typecheck` — TypeScript type check
- `npm test` — Run test suite
- `npx supabase start` — Start local Supabase
- `npx supabase gen types typescript --local > src/types/database.types.ts` — Regenerate DB types

## Architecture Rules (Enforced)
- **Route handlers** → parse request, validate with Zod, return response. NO business logic.
- **Service layer** (`services/`) → all business logic, state machine transitions.
- **Data layer** (`lib/supabase/`) → database queries. RLS enforced on all tables.
- **Money** → always in cents (integers). Use `formatPrice()` for display.
- **Auth** → Supabase JWT. Never trust client-side role checks alone.
- **Contact reveal** → ONLY after `couple_contact_revealed = true` (deposit paid).

## Type Safety (No Compromises)
- `any` is FORBIDDEN — use `unknown` with type guards
- All function parameters and returns must be typed
- Zod for runtime validation on all API inputs
- Database types auto-generated from Supabase (`database.types.ts`)

## What NOT To Do
- Do NOT delete files without explicit confirmation
- Do NOT modify database schemas without migration files
- Do NOT add features not in the current phase (check `AGENTS.md` roadmap)
- Do NOT skip tests for "simple" changes
- Do NOT bypass failing pre-commit hooks
- Do NOT use deprecated libraries or patterns
- Do NOT build: instant booking, vendor CRM, in-app chat, contracts, review system
- Do NOT use `useEffect` for data fetching — use server components
- Do NOT use floating point for money — use integer cents
- Do NOT expose couple contact info before deposit payment

---
> Source: [SardarK942/vendors.io](https://github.com/SardarK942/vendors.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
