## circle

> - `npm run dev` - Start development server

# AGENTS.md - Circle App

## Build & Run Commands
- `npm run dev` - Start development server
- `npm run build` - Build for production (runs prisma generate first)
- `npm run lint` - Run ESLint
- `bun run check` - Run ESLint and TypeScript type checking (MUST run before completing any task)
- `npx prisma generate` - Generate Prisma client
- `npx prisma db push` - Push schema changes to database

## Pre-Completion Checklist
**IMPORTANT**: Before completing any coding task, you MUST run `bun run check` to verify:
1. No ESLint errors or warnings
2. No TypeScript type errors

If errors are found, fix them before considering the task complete.

## Tech Stack
Next.js 15 (App Router), React 19, TypeScript, Prisma (with Accelerate), NextAuth v5, Tailwind CSS 4, Chakra UI, Zod

## Code Style
- Use `@/*` path alias for imports from `src/` (e.g., `import prisma from '@/lib/prisma'`)
- Client components: Add `'use client'` directive at top of file
- API routes: Use NextResponse for responses, auth() for session, return proper HTTP status codes
- Error handling: Try-catch with console.error and appropriate error responses
- Components: Use React.FC with explicit interface for props, prefer memo() for performance
- Naming: camelCase for variables/functions, PascalCase for components/interfaces
- Prisma: Import from `@/lib/prisma`, use typed queries with select/include

## File Structure
- Pages/routes: `src/app/` (App Router)
- API routes: `src/app/api/*/route.ts` with GET/POST/PATCH/DELETE exports
- Components: `src/components/` organized by feature
- Shared utilities: `src/lib/`
- Providers: `src/components/providers/` (SessionProvider, GuestProvider)

## Key Routes
- `/` - Landing page (public)
- `/home` - Authenticated user feed
- `/guest/browse` - Guest browsing mode (public)
- `/auth/login`, `/auth/register` - Authentication (public)
- `/circle/[id]`, `/album/[id]` - Content pages (public read, auth for write)

---
> Source: [nikitalbnv/circle](https://github.com/nikitalbnv/circle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
