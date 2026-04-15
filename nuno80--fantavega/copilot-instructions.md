## fantavega

> - **Type**: Next.js 15 Monorepo-style Application

# Fantavega - AI Agent Guide (Root)

## Project Snapshot
- **Type**: Next.js 15 Monorepo-style Application
- **Core Stack**: React 19, TypeScript, Tailwind CSS, Clerk (Auth), Socket.IO (Real-time).
- **Database**: Turso (Remote LibSQL) / BetterSQLite3 (Local).
- **Architecture**: App Router with distinct Server Components, Client Components, and Server Actions.

## Critical Workflow Rules (MANDATORY)
**After completion of EVERY sub-task**:
1.  **Update State Files**: You MUST generate and provide updated versions of these two JSON files in the same message:
    -   `guide/progetto-attuale/logica-app.json`: Document implemented logic, architectural patterns, and user flows.
    -   `guide/progetto-attuale/struttura-progetto.json`: Map new or modified files.
2.  **Wait for Confirmation**: Do not proceed to the next sub-task until confirmed.

## Root Setup Commands
```bash
pnpm install            # Install dependencies
pnpm run dev            # Start Next.js dev server
pnpm run db:migrate     # Apply Turso schema changes
pnpm run db:generate    # Generate Drizzle/SQL types (if applicable)
```

## Universal Conventions
- **Language**: TypeScript for everything.
- **Auth**: Clerk is the source of truth. User metadata contains `role` ('admin', 'manager').
- **Styling**: Tailwind CSS + Shadcn/UI. No arbitrary CSS files.
- **Strictness**: No `any`. Handle `null`/`undefined` explicitly.

## JIT Index - Documentation Map
**Go to the specific directory's GEMINI.md for detailed context:**

### 1. Application Logic & Routes
- **Path**: `src/app/`
- **Focus**: Pages, Layouts, API Routes, Server Actions.
- **Link**: [src/app/GEMINI.md](src/app/GEMINI.md)

### 2. User Interface
- **Path**: `src/components/`
- **Focus**: UI Components, Shadcn, Hooks, Client State.
- **Link**: [src/components/GEMINI.md](src/components/GEMINI.md)

### 3. Business Logic (The Brain)
- **Path**: `src/lib/db/services/`
- **Focus**: Auction rules, Bidding logic, Penalties, Budget management.
- **Link**: [src/lib/db/services/GEMINI.md](src/lib/db/services/GEMINI.md)

### 4. Database Layer
- **Path**: `database/`
- **Focus**: SQL Schema, Migrations, Seed data.
- **Link**: [database/GEMINI.md](database/GEMINI.md)

### Quick Find Commands
- Search business logic: `rg "class .*Service" src/lib/db/services`
- Find API endpoint: `rg "export async function (GET|POST)" src/app/api`
- Find React component: `rg "export function [A-Z]" src/components`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nuno80)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nuno80)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
