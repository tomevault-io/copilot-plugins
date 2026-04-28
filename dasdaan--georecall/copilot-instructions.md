## georecall

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**GeoRecall** is a web-based quiz platform for GeoGuessr players to memorize visual meta-clues (bollards, road markings, traffic signs, utility poles). The project is currently in the specification phase — all design documents are complete but no implementation code exists yet.

## Tech Stack

- **Framework**: Next.js 14 (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **State**: Zustand (client-side quiz session state only — no user accounts in MVP)
- **Animations**: Framer Motion
- **ORM**: Drizzle ORM with PostgreSQL (via Neon)
- **Validation**: Zod (shared frontend/backend)
- **Image CDN**: Cloudinary
- **Hosting**: Vercel

## Commands

No implementation exists yet. Expected commands once scaffolded:

```bash
npm run dev          # Start development server
npm run build        # Production build
npx drizzle-kit push     # Push schema to DB (development)
npx drizzle-kit migrate  # Apply migrations (production)
npx tsx db/seed.ts       # Seed initial question data
```

## Architecture

### Folder structure (to be created per project spec)

- `/app` — Next.js App Router pages
- `/lib` — Pure business logic (quiz engine, distractor selection, answer validation)
- `/components` — UI components (max ~200 lines each)
- `/types` — Shared TypeScript types
- `/db` — Schema (schema.ts) and seed script (seed.ts)

### Key design decisions

- **Scores are client-side only** (Zustand, not DB) because there are no user accounts in MVP. If accounts are added later, this must migrate to a `SessionScore` table.
- **Spaced repetition**: wrong answer adds +2 to score, correct answer subtracts -1 (min 0). Weighted question selection favors higher-scored questions.
- **Quiz ends** when all question scores reach 0.
- **Logic/UI separation**: quiz engine logic belongs in `/lib`, not inside components.
- **Shared types** go in `/types`, not scattered across files.

### Data model (see `project/GeoRecall_Drizzle_Schema.ts`)

Core entities: `Country`, `Region`, `Category`, `Question`, `DistractorGroup`

- Question ↔ Category is many-to-many
- Image URLs point to Cloudinary: `https://res.cloudinary.com/georecall/image/upload/{category}/{region}/{country}/{country}_{category}_{number}.jpg`
- No raw DB IDs in public URLs

### API routes

- `GET /api/questions` — fetch questions by filters (always validate with Zod, limit results to max 500)
- `GET /api/categories`, `/api/countries`, `/api/regions`

### Page routes (see `project/GeoRecall_Paginaflow_Routing.md`)

`/` → `/quiz-builder` → `/quiz/[sessionId]` → `/results/[sessionId]`  
Also: `/prebuilt`, `/prebuilt/[quizId]`

## Project Documents

Always keep these documents synchronized when making changes:

| Document | Update when... |
|---|---|
| `project/GeoRecall_MVP_Design_Document.md` | New feature added or changed |
| `project/GeoRecall_Design_Brief.md` | Color, font, or animation changed |
| `project/GeoRecall_Paginaflow_Routing.md` | New page or route added |
| `project/GeoRecall_Drizzle_Schema.ts` | DB schema changes |
| `project/GeoRecall_Seed_Data.ts` | Seed data structure changes |

Always update these proactively — never skip an update because the change seems small.

## Code Rules (from .cursorrules)

### Comments
- Explain *why* something is built this way, not *what* it does
- Flag intentional design choices that may look surprising
- No special characters in comments: no `->`, `=>`, `--`, `*`, `**`, `|`
- Write in plain Dutch or plain English — no mixing

### Security
- Validate all query params and request bodies with Zod before use
- Never return more data than needed; cap DB query results
- All user text input must be sanitized and length-limited
- Never do raw SQL — always use Drizzle
- Environment variables via `process.env`, never hardcoded

### When to ask before acting
- New pages or routes not in the routing document
- Schema changes
- Changes affecting multiple files at once
- Uncertainty about what a feature should do

### Always deliver working code
- No imports pointing to non-existent files
- No undefined function calls
- No TypeScript errors you could have solved yourself
- Mark incomplete sections with TODO comments explaining what remains and why

## Design System

Dark mode only. Key colors:
- Background: `#0F1C2E` (Deep Ocean)
- Cards: `#172638` (Atlas Night)
- Primary CTA / correct: `#2ABFAA` (Compass Teal)
- Streak / highlights: `#E8A838` (Map Amber)
- Incorrect / error: `#E85D4A` (Border Red)
- Body text: `#C8C2B4` (Parchment)
- Headings: `#E8E2D8` (Ivory)

Fonts: **Fraunces** (titles, large numbers), **DM Sans** (all other text)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DasDaan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
