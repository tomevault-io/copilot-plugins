## social-ai-poc

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Commands

```bash
# Development
pnpm dev          # Start Next.js dev server on localhost:3000
pnpm build        # Production build
pnpm lint         # ESLint

# Tests (Vitest)
pnpm test              # Run all tests once
pnpm test:watch        # Watch mode
pnpm test:coverage     # Coverage report
pnpm test:ui           # Vitest UI

# Run a single test file
pnpm vitest run src/lib/ai/structured.test.ts

# Prisma
pnpm prisma generate                    # Regenerate client after schema changes
pnpm prisma migrate dev                 # Create + apply migration in dev
pnpm prisma studio                      # GUI for local DB
pnpm prisma db seed                     # Seed (via ts-node)

# Production migrations (Supabase)
pnpm migrate:prod                       # runs scripts/migrate-supabase.sh .env.production
FORCE=true BACKUP=false pnpm migrate:prod   # skip confirmation & backup
```

## Architecture Overview

**Funny Photo Story** is a Next.js 16 (App Router) POC that lets users upload photos into "rooms," then generates AI comic-style stories from them.

### Request flow

1. **Upload** — Client (`UploadClient.tsx`) calls `GET /api/sign` to get a Cloudinary signed upload credential, uploads directly to Cloudinary, then POSTs the result to `POST /api/photos` which persists the photo record via Prisma.
2. **Generate** — `POST /api/story` orchestrates the 3-step AI pipeline: `genBeats → genPanels → genNarrative`. It also optionally calls `captionPhotosOpenAI` to attach captions to photos before generation. Result is saved as a `Story` row with status `READY`.
3. **Share** — `POST /api/story/[id]/share` generates a `shareSlug` and the story is viewable at `/s/[slug]`.

### AI provider layer (`src/lib/ai/`)

The provider system is the core abstraction:

- `providers.ts` — `StoryProvider` interface + `getStoryProvider(kind?)` factory. Provider resolution priority: OpenAI → Anthropic → Mock (based on available API keys).
- `config.ts` — `getCfg(kind)` reads env vars; `getModelForQuality(kind, quality)` maps `fast/balanced/premium` presets to concrete model IDs.
- `provider-openai.ts`, `provider-anthropic.ts`, `provider-mock.ts` — Concrete implementations.
- `structured.ts` — Canonical Zod schemas for `Beat` and `Panel`, plus `safeJson()` for stripping code fences from LLM outputs, and `validateBeats()` / `validatePanels()` validators used by all providers.
- `comic.ts`, `panels.ts`, `text.ts` — Prompt-building utilities.
- `captions-openai.ts` — Optional caption-generation pass (non-fatal if it fails).

### Database (`prisma/schema.prisma`)

PostgreSQL via Supabase. Four models: `User`, `Room`, `Photo`, `Story`.

- Rooms group photos. A `Room` has a short `code` (uppercased).
- `Story.beatsJson` and `Story.panelMap` are `Json` columns storing `Beat[]` and `Panel[]`.
- `Story.status` is `PENDING | PROCESSING | READY | ERROR`.
- `Story.shareSlug` is unique and enables the public `/s/[slug]` route.

### Access control

`src/middleware.ts` gates the entire app behind a `site_access=granted` cookie. Public paths: `/unlock`, `/api/unlock`, `/s/*`, `/api/story/by-slug/*`.

### Routes

| Path | Purpose |
|---|---|
| `/` | Home — lists recent rooms |
| `/u/[roomCode]` | Room workspace — gallery, story controls, uploader |
| `/s/[slug]` | Shared story (public) |
| `/unlock` | Password gate |
| `GET /api/sign` | Cloudinary signed upload params |
| `POST /api/photos` | Persist uploaded photo |
| `POST /api/story` | Generate story (beats → panels → narrative) |
| `GET /api/story/[id]` | Fetch story by ID |
| `POST /api/story/[id]/share` | Assign shareSlug |
| `GET /api/story/by-slug/[slug]` | Fetch story by slug (public) |
| `GET/POST /api/unlock` | Cookie-based access grant |
| `POST /api/rooms` | Create room |

### Path alias

`@/` maps to `src/` (configured in both `tsconfig.json` and `vitest.config.mts`).

### Required environment variables

```
DATABASE_URL                   # PostgreSQL connection string
CLOUDINARY_CLOUD_NAME
CLOUDINARY_API_KEY
CLOUDINARY_API_SECRET
CLOUDINARY_UPLOAD_PRESET       # must be a signed preset
CLOUDINARY_FOLDER              # default: "social-ai-poc"
OPENAI_API_KEY                 # or ANTHROPIC_API_KEY (at least one required for AI)
ANTHROPIC_API_KEY
UNLOCK_PASSWORD                # site access password
```

Optional AI tuning vars: `OPENAI_MODEL`, `OPENAI_MODEL_FAST/BALANCED/PREMIUM`, `ANTHROPIC_MODEL`, `ANTHROPIC_MODEL_FAST/BALANCED/PREMIUM`, `OPENAI_TEMPERATURE`, etc.

### Test setup

Tests use Vitest + jsdom + React Testing Library. `src/test/setup.ts` mocks `next/navigation`, `next/link`, and `next/image`. All API routes use `export const runtime = "nodejs"`.

---
> Source: [eratchev/social-ai-poc](https://github.com/eratchev/social-ai-poc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
