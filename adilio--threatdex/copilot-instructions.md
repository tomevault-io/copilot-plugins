## threatdex

> This file is the single source of truth for any Claude Code agent working on this

# ThreatDex — Claude Code Agent Instructions

This file is the single source of truth for any Claude Code agent working on this
repository. Read it completely before touching any code. Every task handed to you
will reference a phase and issue number from the task list at the bottom.

-----

## 1. Project overview

ThreatDex is a web app that aggregates cyber threat actor intelligence from public
CTI feeds (MITRE ATT&CK, ETDA, AlienVault OTX, MISP, OpenCTI) and renders each
actor as an interactive "trading card" — flippable front/back with stats, TTPs,
campaigns, tools, and references.

**Tagline:** Know your adversaries, card by card.

-----

## 2. Project structure

```
threatdex/
├── app/                      # React Router v7 application
│   ├── components/           # Card UI components
│   ├── lib/
│   │   ├── supabase.client.ts  # Browser Supabase client
│   │   └── supabase.server.ts  # Server-side Supabase client
│   ├── routes/
│   │   ├── _index.tsx        # Home page — card grid + search/filters
│   │   └── actors.$id.tsx    # Actor detail page
│   ├── schema/
│   │   └── index.ts          # Zod schemas + TypeScript types (canonical data model)
│   ├── app.css               # Tailwind CSS v4 + brand design system
│   ├── root.tsx
│   └── routes.ts
├── workers/                  # TypeScript data ingestion scripts (run via tsx)
│   ├── mitre-sync.ts         # MITRE ATT&CK STIX bundle ingestion
│   ├── etda-sync.ts          # ETDA APT scraper
│   ├── otx-sync.ts           # AlienVault OTX connector
│   ├── image-gen.ts          # AI hero image generation
│   └── shared/               # Shared utilities
│       ├── supabase.ts       # Supabase admin client for workers
│       ├── models.ts         # Shared TypeScript data models
│       ├── dedup.ts          # Alias deduplication + actor merge logic
│       └── rarity.ts         # Rarity tier + threat level computation
├── supabase/
│   └── migrations/           # PostgreSQL schema + RLS policies
├── tests/
│   ├── components/           # Vitest component tests
│   ├── e2e/                  # Playwright smoke tests
│   └── workers/              # Worker unit tests
├── .github/
│   └── workflows/
│       ├── ci.yml            # Test + lint on push/PR
│       └── sync.yml          # Nightly data sync cron
├── docs/
│   ├── API.md
│   ├── DATA_SOURCES.md
│   └── ARCHITECTURE.md
├── .env.example
├── CLAUDE.md                 ← this file
├── CONTRIBUTING.md
├── SECURITY.md
├── netlify.toml              # Netlify deployment (edge SSR)
└── README.md
```

-----

## 3. Tech stack

| Layer           | Technology                                                      |
|-----------------|-----------------------------------------------------------------|
| Frontend        | React Router v7 (Vite SSR), TypeScript, Tailwind CSS v4        |
| Database        | PostgreSQL 15 via Supabase (managed, auto-REST)                 |
| Data access     | Supabase JS SDK (`@supabase/supabase-js`) in React Router loaders |
| Workers         | TypeScript scripts executed via `tsx` in GitHub Actions         |
| Package manager | pnpm (single `package.json`, no workspaces)                     |
| Testing         | Vitest (unit), Playwright (e2e)                                 |
| Linting         | ESLint + TypeScript-ESLint                                      |
| CI              | GitHub Actions                                                  |
| Hosting         | Netlify (edge SSR via `@netlify/vite-plugin-react-router`)      |

-----

## 4. Brand + design system

Apply these values consistently across all frontend work.

```
--color-wiz-blue:       #0254EC   /* primary CTAs, nav */
--color-purplish-pink:  #FFBFFF   /* accents, highlights */
--color-cloudy-white:   #FFFFFF   /* backgrounds, card surfaces */
--color-serious-blue:   #00123F   /* page background, card borders */
--color-blue-shadow:    #173AAA   /* secondary nav, card frames */
--color-sky-blue:       #6197FF   /* links, tags */
--color-light-sky-blue: #978BFF   /* subtle accents */
--color-pink-shadow:    #C64BA4   /* hover states */
--color-vibrant-pink:   #FF0BBE   /* rarity badges, "Dex" logotype */
--color-frosting-pink:  #FFBFD6   /* soft backgrounds */
--color-surprising-yellow: #FFFF00 /* MYTHIC tier glow, warnings */
```

Typography: monospace for all data/stat fields, a clean sans-serif for body copy.
Overall feel: dark navy base, vivid blue/pink accents — Wiz-style, optimistic security.

-----

## 5. Data model (canonical)

All ingestion workers and UI components must conform to this schema.
The authoritative TypeScript definition lives in `app/schema/index.ts`.

```typescript
interface ThreatActor {
  id: string                    // slug, e.g. "apt28"
  canonicalName: string
  aliases: string[]
  mitreId?: string              // e.g. "G0007"
  country?: string
  countryCode?: string          // ISO 3166-1 alpha-2
  motivation: Motivation[]      // "espionage" | "financial" | "sabotage" | "hacktivism" | "military"
  threatLevel: number           // 1–10
  sophistication: Sophistication
  firstSeen?: string            // YYYY
  lastSeen?: string             // YYYY
  sectors: string[]
  geographies: string[]
  tools: string[]
  ttps: TTPUsage[]
  campaigns: Campaign[]
  description: string
  tagline?: string
  rarity: Rarity                // "MYTHIC" | "LEGENDARY" | "EPIC" | "RARE"
  imageUrl?: string
  imagePrompt?: string
  sources: SourceAttribution[]
  tlp: "WHITE" | "GREEN"
  lastUpdated: string           // ISO 8601
}

interface TTPUsage {
  techniqueId: string           // e.g. "T1566"
  techniqueName: string
  tactic: string
}

interface Campaign {
  name: string
  year?: string
  description: string
}

interface SourceAttribution {
  source: "mitre" | "etda" | "otx" | "misp" | "opencti" | "manual"
  sourceId?: string
  fetchedAt: string
  url?: string
}

type Motivation = "espionage" | "financial" | "sabotage" | "hacktivism" | "military"
type Sophistication = "Low" | "Medium" | "High" | "Very High" | "Nation-State Elite"
type Rarity = "MYTHIC" | "LEGENDARY" | "EPIC" | "RARE"
```

-----

## 6. Environment variables

Never commit secrets. Always read from environment. The full list lives in `.env.example`.

```bash
# Supabase (required)
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=eyJ...          # Public, browser-safe
SUPABASE_SERVICE_KEY=eyJ...       # Private, server-side + workers only

# CTI sources (optional — feature-flag gracefully if missing)
OTX_API_KEY=
OPENAI_API_KEY=
MISP_URL=
MISP_API_KEY=
OPENCTI_URL=
OPENCTI_API_KEY=
```

If an optional key is missing, the relevant worker should log a warning and skip
gracefully — never crash the process.

-----

## 7. Data access pattern

Data is stored in Supabase (PostgreSQL). The frontend accesses it via React Router
loaders using the Supabase JS SDK — there is no separate REST API server.

### Route loader pattern

```typescript
// app/routes/_index.tsx
import { createServerClient } from "~/lib/supabase.server"

export async function loader({ request }: LoaderFunctionArgs) {
  const supabase = createServerClient(request)
  const { data, error } = await supabase
    .from("actors")
    .select("*")
    .order("threat_level", { ascending: false })
    .limit(20)
  // ...
}
```

### Supabase table: `actors`

Columns mirror the `ThreatActor` schema. JSON columns store `ttps`, `campaigns`,
`sources`, `aliases`, `tools`, `sectors`, `geographies`, `motivation`.

### Workers write via service key

Workers use `SUPABASE_SERVICE_KEY` to bypass RLS and upsert actor records:

```typescript
// workers/shared/supabase.ts
import { createClient } from "@supabase/supabase-js"
export const supabase = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_KEY!
)
```

### Admin sync

Manual data syncs are triggered by running workers directly:

```bash
pnpm workers:mitre    # MITRE ATT&CK
pnpm workers:etda     # ETDA
pnpm workers:otx      # AlienVault OTX
pnpm workers:image    # AI image generation
pnpm workers:all      # all sources in sequence
```

The nightly cron in `.github/workflows/sync.yml` runs these same commands.

-----

## 8. Git workflow

All changes go directly to `main`. No feature branches, no PRs.

### Step-by-step for every task

```bash
# 1. Pull latest main
git checkout main
git pull origin main

# 2. Do the work. Use conventional commits:
#    feat: add ETDA scraper worker
#    fix: correct alias deduplication logic
#    chore: add Supabase migration for actors table
#    test: add Vitest coverage for rarity computation
#    docs: update DATA_SOURCES.md with OTX connector

# 3. Before committing, run:
pnpm lint && pnpm test

# 4. Stage relevant files, commit with a subject + body, push
git add <files>
git commit -m "type: subject

Body explaining what changed and why."
git push origin main
```

-----

-----

## 10. Testing requirements

- **Workers:** Each sync worker needs a Vitest test that mocks the upstream fetch
  and verifies the normalised output matches the `ThreatActor` schema.
- **Frontend:** New page routes need a Playwright smoke test. New components need
  a Vitest unit test.
- **Do not merge if tests are red.**

Run all checks before opening a PR:

```bash
pnpm lint         # ESLint
pnpm test         # Vitest unit tests
pnpm test:e2e     # Playwright e2e (requires running app)
```

-----

## 11. Task list — phased breakdown

Each task below is a self-contained GitHub Issue. Hand any single item to an agent.
Items within a phase can run in parallel unless marked with ⛔ (blocked by another).

-----

### Phase 1 — Foundation ✅ Complete

*Goal: any engineer can clone and run the full stack in under 5 minutes.*

| Issue | Title                                                  | Status   |
|-------|--------------------------------------------------------|----------|
| #1    | Init project with React Router v7 + Tailwind v4 + pnpm | ✅ Done  |
| #2    | Set up Supabase project + migrations for actors table  | ✅ Done  |
| #3    | Create .env.example with all required vars             | ✅ Done  |
| #4    | Add Supabase RLS policies                              | ✅ Done  |
| #5    | Scaffold home page + actor detail route                | ✅ Done  |
| #6    | Add GitHub Actions CI workflow (lint + test)           | ✅ Done  |

-----

### Phase 2 — Data ingestion ✅ Complete

*Goal: real threat actor data flowing into the database.*

| Issue | Title                                          | Status  |
|-------|------------------------------------------------|---------|
| #7    | MITRE ATT&CK STIX bundle ingestion worker      | ✅ Done |
| #8    | ETDA scraper worker                            | ✅ Done |
| #9    | AlienVault OTX connector                       | ✅ Done |
| #10   | Alias deduplication + actor merge logic        | ✅ Done |
| #11   | Rarity tier + threat level computation         | ✅ Done |
| #12   | Add nightly sync GitHub Actions cron           | ✅ Done |

-----

### Phase 3 — Frontend

*Goal: the card UI is live and pulling from real data.*

| Issue | Title                                              | Branch slug         | Notes                    |
|-------|----------------------------------------------------|---------------------|--------------------------|
| #13   | Card front + back components with flip animation   | `card-flip`         | ⛔ needs Phase 2          |
| #14   | Home page — card grid with search + filters        | `home-page`         | ⛔ needs #13              |
| #15   | Actor detail page /actors/:id                      | `actor-detail-page` | ⛔ needs #13              |
| #16   | Card download — export as PNG via html2canvas      | `card-download`     | ⛔ needs #13              |
| #17   | Add `<meta>` tags for actor pages (SEO)            | `actor-seo`         | ⛔ needs #15              |
| #18   | Mobile responsive layout                           | `mobile-layout`     | ⛔ needs #14 #15          |

-----

### Phase 4 — Image generation

*Goal: every actor has a unique AI-generated hero image.*

| Issue | Title                                                  | Branch slug            | Notes                                       |
|-------|--------------------------------------------------------|------------------------|---------------------------------------------|
| #19   | Image prompt template builder from ThreatActor fields  | `image-prompt-builder` |                                             |
| #20   | Image generation worker (OpenAI DALL-E)                | `image-gen-worker`     | Feature-flagged off if no `OPENAI_API_KEY`  |
| #21   | Image storage via Supabase Storage                     | `image-storage`        | ⛔ needs #20                                 |
| #22   | Wire generated images into card front hero area        | `card-hero-image`      | ⛔ needs #21 #13                             |

-----

### Phase 5 — Polish + launch

*Goal: production-ready, publicly shareable.*

| Issue | Title                                             | Branch slug         | Notes              |
|-------|---------------------------------------------------|---------------------|--------------------|
| #23   | Add Playwright e2e smoke tests for core flows     | `e2e-tests`         | ⛔ needs #14 #15    |
| #24   | Deploy to Netlify (edge SSR)                      | `deploy`            | ⛔ needs Phase 3    |
| #25   | Add optional MISP connector                       | `misp-connector`    | ⛔ needs #10        |
| #26   | Add optional OpenCTI connector                    | `opencti-connector` | ⛔ needs #10        |

-----

## 12. How to hand a task to an agent

Copy this block, fill in the issue number and title, and paste it to Claude Code:

```
You are working on the ThreatDex repository.
Read CLAUDE.md fully before writing any code.

Your task: Issue #{N} — {TITLE}

Acceptance criteria:
- {paste the acceptance criteria from the issue}

Workflow:
1. Pull latest main
2. Implement the task per the spec in CLAUDE.md
3. Run pnpm lint && pnpm test — fix any failures
4. Commit directly to main with a conventional commit message (subject + body)
   and push

Stay within the scope of this issue.
If you discover work that belongs in a different issue, open a GitHub Issue
for it — do not do that work in this commit.
```

-----

## 13. Definition of done

A task is done when:

- [ ] All new code has tests
- [ ] `pnpm lint && pnpm test` passes
- [ ] Changes committed directly to `main` with a conventional commit message
- [ ] No secrets are committed
- [ ] No `console.log` debug statements left in

---
> Source: [adilio/threatdex](https://github.com/adilio/threatdex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
