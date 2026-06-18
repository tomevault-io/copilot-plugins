## tms

> A Bengali-language tailoring shop POS/management system built with Next.js App Router and Supabase (PostgreSQL) database.

# Tailor Shobai - Tailoring Shop Management System

## Project Overview
A Bengali-language tailoring shop POS/management system built with Next.js App Router and Supabase (PostgreSQL) database.

## Tech Stack
- **Framework**: Next.js 15+ (App Router, `'use client'` pages)
- **Language**: TypeScript
- **Auth**: better-auth (PostgreSQL-backed via Supabase)
- **Database**: Supabase (PostgreSQL with JSONB columns)
- **UI**: Tailwind CSS + shadcn/ui (Radix primitives)
- **Icons**: lucide-react
- **Charts**: Recharts
- **State**: React Context (DataContext, ThemeContext, LanguageContext)
- **Forms**: react-hook-form + zod
- **Data Fetching**: TanStack React Query
- **PDF**: html2pdf.js

## Architecture

### Directory Structure
```
src/
├── app/api/          # API routes (Next.js Route Handlers)
├── app/(pages)/      # App Router pages
├── views/            # Page-level view components
├── components/       # Shared components (Layout, NavLink, ui/)
├── context/          # React contexts (Data, Theme, Language)
├── hooks/            # Custom hooks
├── lib/              # Core utilities (auth, cloud-db, supabase, utils)
├── types/            # TypeScript interfaces
```

### Database Pattern
- `getGlobalSupabase()` → Supabase client from env vars
- `getCloudDb(shopId)` → Returns global Supabase client
- All data stored in Supabase with JSONB columns (auto-parsed)
- Auth uses PostgreSQL via `DATABASE_URL` (Better Auth + pg driver)

### Environment Variables Required
- `DATABASE_URL` — PostgreSQL connection string (for Better Auth)
- `SUPABASE_URL` — Supabase project URL
- `SUPABASE_SERVICE_ROLE_KEY` — Supabase service role key
- `BETTER_AUTH_SECRET` — Auth secret
- `BETTER_AUTH_URL` — App base URL

### API Route Pattern
All routes in `src/app/api/` follow this pattern:
1. Authenticate via `auth.api.getSession()`
2. Get `shopId` from user via `getShopId(userId)`
3. Use `getCloudDb(shopId)` or `getGlobalSupabase()` for database operations

### UI Conventions
- Bengali (বাংলা) as primary language
- All labels use `t()` from LanguageContext
- Theme-aware with CSS custom properties
- Mobile-responsive with sidebar (desktop) + bottom nav (mobile)
- shadcn/ui components in `src/components/ui/`

## Commands
- `npm run dev` — Start dev server
- `npm run build` — Production build
- `npm run lint` — ESLint

---
> Source: [mdnhs/tms](https://github.com/mdnhs/tms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
