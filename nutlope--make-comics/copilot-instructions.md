## make-comics

> This app features AI generation of comic books using AI Image models with user references being included in the final outputs making it possible to create custom comic books with different styles and with real characters in them. It uses Together AI inference to run these AI models in this Next.js app.

# AGENTS.md

This app features AI generation of comic books using AI Image models with user references being included in the final outputs making it possible to create custom comic books with different styles and with real characters in them. It uses Together AI inference to run these AI models in this Next.js app.

## Commands
- `pnpm dev` - Start development server
Never execute this command always consider the "dev" server as already running, if inside CI then just consider build and lint. Locally dev server is run from the user.
- `pnpm build` - Production build
- `pnpm lint` - Run ESLint
- `pnpm drizzle-kit push` - Push DB schema changes, always remember to do this when doing schema changes with drizzle to prevent mismatch between schema and live db.
- `pnpm drizzle-kit generate` - Generate migrations

## Architecture
- **Framework**: Next.js 16 (App Router) with React 19, TypeScript
- **Auth**: Clerk (`@clerk/nextjs`)
- **Database**: Neon Postgres with Drizzle ORM (`lib/schema.ts`, `lib/db.ts`)
- **Storage**: AWS S3 for images (`lib/s3-upload.ts`)
- **AI**: Together AI for comic generation (`together-ai`)
- **Rate Limiting**: Upstash Redis (`lib/rate-limit.ts`)
- **UI**: Radix primitives + shadcn/ui (`components/ui/`), Tailwind CSS v4

## Code Style
- Use `@/*` path alias for imports (e.g., `@/lib/utils`, `@/components/ui/button`)
- React Server Components by default; add `"use client"` only when needed
- Export types from schema: `Story`, `NewStory`, `Page`, `NewPage`
- Use Zod for validation, react-hook-form for forms
- API routes in `app/api/*/route.ts`, server actions in `lib/db-actions.ts`
- Component structure: `components/ui/` (primitives), `components/editor/`, `components/landing/`

---
> Source: [Nutlope/make-comics](https://github.com/Nutlope/make-comics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
