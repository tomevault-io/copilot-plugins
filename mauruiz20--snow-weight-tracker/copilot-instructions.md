## snow-weight-tracker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Snow Weight Tracker is a weight tracking web app for friends preparing for a snowboarding trip to Chile (August 1, 2026). Users track weight loss progress and compete on a leaderboard.

## Tech Stack

- **Next.js 16** with App Router (React 19)
- **Supabase** for database (PostgreSQL) and realtime subscriptions
- **Tailwind CSS v4** for styling
- **Biome** for linting and formatting (not ESLint for day-to-day use)
- **TypeScript**

## Commands

```bash
npm run dev          # Start development server
npm run build        # Production build
npm run lint         # Lint with Biome
npm run lint:fix     # Auto-fix lint issues
npm run format       # Format with Biome
npm run check        # Run all Biome checks (lint + format)
```

### Supabase

```bash
npx supabase db push              # Push migrations to remote
npx supabase start                # Start local Supabase
npx supabase migration new <name> # Create new migration
```

Migrations are in `supabase/migrations/` and should be numbered sequentially (001_, 002_, etc.).

## Architecture

### Data Flow

1. **API Layer** (`src/lib/api/`) - All Supabase queries are centralized here
2. **Custom Hooks** (`src/hooks/`) - React hooks consume the API layer and manage state
3. **Components** - Use hooks for data, never call API directly

### Key Patterns

- **Realtime subscriptions**: `useRealtimeSubscription` hook wraps Supabase realtime for live updates
- **Typed database**: All types are in `src/types/database.types.ts`, derived from Supabase schema
- **Constants**: App-wide constants (routes, breakpoints, chart config) are in `src/lib/constants.ts`
- **Audit logging**: Updates/deletes are logged via `src/lib/supabase/audit.ts`

### Database Schema

Two main tables with a view for rankings:
- `participants` - User profiles (name, age, gender, height, initial_weight, target_weight)
- `weight_records` - Weight entries linked to participants
- `participant_rankings` (view) - Computed rankings with weight loss percentages, BMI, etc.

### Component Organization

- `src/components/ui/` - Reusable UI primitives (Button, Input, Select, etc.)
- `src/components/participants/` - Participant-specific components
- `src/components/stats/` - Statistics, charts, leaderboard
- `src/components/weight/` - Weight record forms and lists

### Environment Variables

Required in `.env.local`:
```
NEXT_PUBLIC_SUPABASE_URL=<supabase-url>
NEXT_PUBLIC_SUPABASE_ANON_KEY=<supabase-anon-key>
```

## Code Style

- Biome enforces: single quotes, no semicolons, 2-space indent, 100 char line width
- Use arrow functions (`useArrowFunction` rule enabled)
- Spanish language for user-facing strings (locale: es-ES)
- Import UI components from `@/components/ui` barrel export

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mauruiz20) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
