## japaneasy

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Japaneasy is a Japanese language learning platform built with Next.js 16 (App Router) and React 19. It uses a spaced repetition system (SRS) for JLPT vocabulary study, with gamified progression through levels N5 to N1. Includes an admin panel for managing courses, lessons, and words.

## Commands

```bash
npm run dev          # Start dev server (localhost:3000)
npm run build        # Production build
npm run start        # Start production server
npm run test         # Run tests with Bun
npm run test:watch   # Run tests in watch mode

# Database (requires PostgreSQL via Docker)
npm run db:start              # Start PostgreSQL 17.0 container
npm run db:stop               # Stop PostgreSQL container
npm run db:generate           # Generate Drizzle migrations from schema
npm run db:migrate            # Apply migrations to database
npm run db:push               # Push schema directly (skips migration files)
npm run db:studio             # Open Drizzle Studio visual browser
npm run db:seed               # Seed database (bun run scripts/seed.ts)
npm run db:verify-indexes     # Verify database indexes
```

## Architecture

### Routing & Layout Groups

The app uses Next.js App Router with route groups for layout separation:
- `src/app/(main)/` — Protected pages: dashboard, study (learn/review), dictionary, hiragana, katakana, settings
- `src/app/admin/` — Admin panel: CRUD for courses, lessons, units, words (separate layout)
- `src/app/auth/` — Public auth pages: signin, signup, password-reset

Each group has its own layout, enabling different navigation and styling per context.

### Authentication

Better Auth with Drizzle adapter (`src/lib/auth.ts`). Email/password auth enabled. Better Auth manages its own tables (`user`, `session`, `account`, `verification`) defined in `src/drizzle/schema/auth-schema.ts`. Application-specific user data (e.g., `userProgress`) is separate and references Better Auth's user ID.

Auth API catch-all route: `src/app/api/auth/[...all]/route.ts`.

### Database

- **ORM**: Drizzle ORM with PostgreSQL (`postgres` driver), max 10 connections
- **Schema**: `src/drizzle/schema/` — split into multiple files, exported via `index.ts`
  - `auth-schema.ts` — Better Auth tables (user, session, account, verification)
  - `enums.ts` — PostgreSQL enums: `jlptLevelEnum` (N5–N0), `cardStatusEnum`
  - `course.ts` — Courses and lessons tables with relationships
  - `user-progress.ts` — Three tables: `userProgress` (streak, kana quiz flags), `userCourseProgress` (per-course tracking), `userWordProgress` (SRS card state: interval, easeFactor, repetitions, nextReviewDate, streakCount)
  - `word.ts` — Vocabulary master data (kanji, kana, romaji, english, partOfSpeech, wordGroup, level, displayOrder)
  - `review.ts` — Audit trail of all review events
- **Client**: `src/drizzle/index.ts` — exports singleton `db` instance
- **Migrations**: `src/drizzle/migrations/`
- **Config**: `drizzle.config.ts` — points to schema, uses `DATABASE_URL` from `.env`
- **Docker**: `docker-compose.yml` — PostgreSQL 17.0, reads `DB_NAME`, `DB_USER`, `DB_PASSWORD` from `.env`

### Server Architecture

Clean three-layer separation under `src/server/`:

- **`actions/`** — Server actions (Next.js `'use server'`) consumed by the frontend:
  - `actions.ts` — `getNewCards`, `completeLearnSession`, `getDueCards`
  - `auth-actions.ts` — `getUser`
  - `settings-actions.ts` — `updateName`, `changePassword`
  - `admin-actions.ts` — Full CRUD for courses, words, and lessons
- **`queries/`** — Read-only database operations (includes `admin-queries.ts`)
- **`mutations/`** — Write database operations (includes `admin-mutations.ts`)

### SRS Algorithm

Located in `src/lib/srs/`:
- `algorithm.ts` — Core logic: `calculateNextReview(card, wasCorrect)` and `shouldAddNewWords(correctCount, totalCount)`
- `constants.ts` — Configuration: learning intervals `[1, 3, 7]` days, graduating interval 7, ease factor range `1.3–3.0`, mastery threshold 60 days, accuracy threshold 80%, batch size 5 words
- `__tests__/algorithm.test.ts` — Comprehensive Bun test suite

**Card state machine**: `new → learning → reviewing → mastered`. Incorrect answers reset to learning. Ease factor decreases by 0.2 on failure.

### Additional Libraries

- `src/lib/word-of-the-day.ts` — Word of the day logic
- `src/lib/hiragana-data.ts`, `src/lib/katakana-data.ts` — Kana character data for practice
- `src/lib/validations/admin.ts` — Zod schemas for admin forms
- `src/lib/types.ts` — Shared types (WordOfTheDay, ActionResponse, UnitProgress)

### UI Components

shadcn/ui pattern (new-york style) with components in `src/components/ui/`. Uses:
- Class Variance Authority (CVA) for component variants
- Radix UI primitives for accessibility (including `collapsible.tsx`)
- Lucide React for icons
- React Hook Form + Zod for form validation

shadcn config is in `components.json`. Add new components with `npx shadcn@latest add <component>`.

**Custom components** in `src/components/`:
- `progress-bar.tsx`, `progress-card.tsx` — Progress display components
- `unit-card.tsx` — Collapsible JLPT level card with integrated progress bars and CTA
- `sidebar.tsx`, `sidebar-item.tsx` — Main app navigation
- `word-of-the-day.tsx`, `quick-practice-card.tsx`, `notifications-card.tsx` — Dashboard widgets
- `mascot.tsx`, `logo.tsx` — Branding components
- `admin/` — Admin UI: sidebar, CRUD forms (course, lesson, unit, word), data-table, pagination, delete-dialog
- `layout/` — `feed-wrapper.tsx`, `sticky-wrapper.tsx`

### Styling

- Tailwind CSS 4 with PostCSS
- OKLch color space for all design tokens (defined as CSS variables in `src/app/globals.css`)
- Light/dark mode via `.dark` class selector (next-themes)
- Custom brand colors: `--orange-light`, `--orange-medium`, `--orange-dark`, `--peach`, `--cream`
- Three fonts: Geist (sans), Geist Mono, Cherry Bomb One (display/branding)

### Scripts

`scripts/` contains seed data and utilities:
- `seed.ts` — Database seeder (run via `npm run db:seed`)
- `verify-indexes.ts` — Database index verification
- `sort_by_frequency.py` — Python script to sort words by frequency
- `verify_words.py` — Python script to verify word data
- CSV files — Vocabulary data for seeding (N5 words)

### Path Aliases

`@/*` maps to `./src/*` (configured in tsconfig.json).

## Code Style

- Prettier with `@ianvs/prettier-plugin-sort-imports` and `prettier-plugin-tailwindcss`
- Single quotes, semicolons, `es5` trailing commas, arrow parens `avoid`, print width 100
- Import order: react > next > third-party > workspace > drizzle > types > config > lib > hooks > auth > components/ui > components > app > relative
- ESLint: `next/core-web-vitals` + `next/typescript` (unused vars rule disabled)
- React Compiler enabled in `next.config.ts`
- Tests use the Bun test runner (run with `bun test` or `npm run test`)

---
> Source: [tabuhana/japaneasy](https://github.com/tabuhana/japaneasy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
