## exam-assessment

> A full-stack monorepo for an exam assessment platform.

# Exam Assessment Platform - Model Context Protocol

## Project Overview
A full-stack monorepo for an exam assessment platform.

## Tech Stack
- **Framework**: Monorepo using `pnpm` workspaces
- **Frontend**: Next.js 14+ (Portal & Admin)
- **Backend Services**: Hono on Cloudflare Workers (Auth, Papers, etc.)
- **Database**: PostgreSQL (Neon) with Drizzle ORM
- **Styling**: SCSS Modules & Ant Design
- **Shared Packages**: 
  - `@assessment/db`: Database schema and migrations
  - `@assessment/types`: Shared TypeScript interfaces
  - `@assessment/utils`: Shared utilities (hashing, error handling)

## Key Project Structure
- `apps/portal`: Student-facing application
- `apps/admin`: Admin-facing application
- `services/*`: Microservices (Auth, Papers, UI-Gateway)
- `packages/db`: Drizzle schemas and seed scripts
- `packages/types`: Shared type definitions
- `packages/utils`: Shared utility functions

## Developer Rules & Patterns
1. **Types**: Always use shared types from `@assessment/types`.
2. **Database**: All DB interactions must use Drizzle ORM.
3. **API**: Backend services use Hono with Zod validation.
4. **Errors**: Always use the custom `ErrorCode` enum from `@assessment/utils`.
5. **Exam Feature**: 
   - Dynamic paper duration is stored in the `papers` table (`durationMinutes`).
   - Exam sessions use a dynamic `expiresAt` calculated at start.
   - Scoring is handled server-side in `papers-svc` using matching correct option IDs.

## Common Operations
- `pnpm dev`: Start all apps/services in development mode
- `pnpm db:seed`: Seed the database with sample papers/questions
- `pnpm build`: Build all workspace packages

---
> Source: [mkskaushalya/exam-assessment](https://github.com/mkskaushalya/exam-assessment) — distributed by [TomeVault](https://tomevault.io/claim/mkskaushalya).
<!-- tomevault:4.0:copilot_instructions:2026-04-18 -->
