## trading-history-ai

> Siempre responde en español.

Siempre responde en español.
Always use Context7 MCP when I need library/API documentation, code generation, setup or configuration steps without me having to explicitly ask.

# Project Context & Rules

## Tech Stack
- **Framework**: Next.js 14+ (App Router).
- **Language**: TypeScript (Strict mode).
- **Backend/DB**: Supabase (PostgreSQL).
- **Auth**: Supabase Auth (SSR w/ PKCE).
- **Styling**: Tailwind CSS.
- **Components**: Shadcn/ui (Radix UI + Tailwind).
- **State Management**: React Server Components (Server state) + Zustand (Client state only if complex).

## Architecture & Patterns
- **Directory Structure**: Feature-based (`src/features/auth`, `src/features/trades`).
    - Inside features: `components/`, `actions.ts` (Server Actions), `types.ts`, `hooks/`.
- **Data Fetching**: Prefer React Server Components (RSC) accessing Supabase directly.
- **Mutations**: Use Server Actions for all writes. Validate inputs with Zod.
- **Client Components**: Use 'use client' only when interactivity is needed (hooks, event listeners).
- **Supabase**:
    - Use `@supabase/ssr` package.
    - Follow the "CookieAuth" pattern for middleware and clients.
    - Row Level Security (RLS) is MANDATORY for all tables.

## Coding Standards
- **Naming**: `kebab-case` for files/folders, `PascalCase` for components, `camelCase` for functions/vars.
- **Types**: Define interfaces for all DB tables (generate with Supabase CLI if possible) and props.
- **Error Handling**: Use `try/catch` in Server Actions and return standard error objects `{ success: boolean, error?: string, data?: T }`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jcamilodev1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
