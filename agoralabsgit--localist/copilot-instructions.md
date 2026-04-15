## localist

> AI-assisted social life planner for Buenos Aires. PWA.

# Localist — Cursor Rules

## Project
AI-assisted social life planner for Buenos Aires. PWA.

## Stack
- Next.js 14+ (App Router), TypeScript, Tailwind CSS, shadcn/ui
- Supabase (Postgres, Auth, RLS)
- Stripe (subscriptions)
- Vercel (deploy)

## Conventions
- Use `src/` directory (app, components, lib, types, hooks)
- Server components by default; add "use client" only when needed
- Supabase client via `@supabase/ssr` (not deprecated `auth-helpers`)
- Use `@/` import alias
- Tailwind for all styling — no CSS modules
- shadcn/ui components in `src/components/ui/`
- Custom components in `src/components/`
- Types in `src/types/`
- Supabase helpers in `src/lib/supabase/`
- API routes in `src/app/api/`
- DB migrations in `supabase/migrations/`
- Scripts (ingestion, seeds) in `scripts/`

## Code Style
- Functional components, named exports
- Prefer `interface` over `type` for object shapes
- Use Supabase generated types from `src/types/database.ts`
- Error handling: try/catch in server actions, error boundaries in UI
- Keep components small and composable

## DB Tables (MVP)
users, user_preferences, venues, highlights, ratings, saved_items, ingestion_jobs
(events table comes in Phase 2)

## Current Phase
MVP: Highlights (Google Places → venues + highlights), UI shell, auth, save/rate

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AgoraLabsGit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
