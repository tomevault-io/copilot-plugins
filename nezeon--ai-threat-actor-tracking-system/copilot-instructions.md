## ai-threat-actor-tracking-system

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HivePro Threat Intel AI — a full-stack Threat Intelligence Platform (TIP) that uses Google Gemini 3 Flash to generate real-time, validated threat actor profiles. Features include a two-pass AI architecture (research + structuring), a 4-step post-generation validation pipeline (trusted data override, MITRE ATT&CK enrichment, NVD CVE verification, URL validation), client-side RAG via PDF/CSV/XLSX uploads, granular section refresh, alias cross-validation, generation logging, context-aware chat, live cyber news, and Excel export.

## Commands

```bash
# Install all dependencies (root + client + server)
npm run setup

# Run both client and server concurrently (dev mode)
npm run dev

# Run only the client (Vite on port 3000)
npm run dev:client

# Run only the server (tsx watch on port 3001)
npm run dev:server

# Build client for production
npm run build

# Build server TypeScript
cd server && npm run build

# Run production server (serves client build)
NODE_ENV=production npm start

# TypeScript type-check (server)
cd server && npx tsc --noEmit

# Run profile accuracy tests
npx tsx server/src/tests/validateProfiles.ts

# Test single actor
npx tsx server/src/tests/validateProfiles.ts "APT29"
```

## Prerequisites

- **Node.js** 18+
- **PostgreSQL** 14+ with a database named `threatintel`
- **`.env`** file in project root (copy from `.env.example`):
  - `GEMINI_API_KEY` — Google Gemini API key
  - `DATABASE_URL` — PostgreSQL connection string (default: `postgresql://localhost:5432/threatintel`)
  - `PORT` — Server port (default: 3001)
  - `CLIENT_ORIGIN` — CORS origin (default: `http://localhost:3000`)

The server auto-creates all six tables on startup via `initializeDatabase()` and seeds initial threat actor data if the DB is empty.

## Architecture

**Monorepo with three packages:**

```
/                   Root — concurrently orchestrates client + server
├── client/         React 19 SPA (Vite, TypeScript, Tailwind CSS)
├── server/         Express API (TypeScript, tsx, PostgreSQL via pg)
└── shared/         Shared TypeScript types (ThreatActor, CVE, GenerationLog, etc.)
```

### Client (`client/`)

- **Vite** dev server on port 3000, proxies `/api` requests to the Express server (see `vite.config.ts`)
- Path alias: `@/` maps to `client/src/`
- `client/src/services/apiService.ts` — single HTTP client wrapping all `/api` endpoints
- Views are toggled via `ViewState` enum: ThreatActors, Chat, LiveFeed, TrustedSources
- No routing library — `App.tsx` switches panels based on state

### Server (`server/`)

- Entry: `server/src/index.ts` — Express app with helmet, cors, morgan
- `server/src/env.ts` — loads `.env` from project root via dotenv (imported first)
- ESM throughout — all local imports use `.js` extensions (TypeScript compiles to ESM)
- Route structure: `routes/` → `controllers/` → `services/` + `models/`

**Key services:**
- `geminiService.ts` — core AI pipeline. Two-pass architecture (research with web search → structured JSON extraction). Post-generation 4-step pipeline: trusted CSV override → MITRE enrichment → NVD CVE verification → URL validation → minimum sources. Returns `{ profile, log }` where log tracks all pipeline steps and sources
- `mitreService.ts` — fetches MITRE ATT&CK enterprise STIX data from GitHub, caches in-memory for 24h, provides alias/first_seen lookups, fuzzy name matching
- `aliasValidationService.ts` — cross-validates aliases against MITRE dataset. Rejects aliases with qualifiers, malware/tool names, and aliases attributed to different actors
- `urlValidation.ts` — validates source URLs via HEAD/GET requests (5s timeout), tiered whitelist system (Tier 0: MITRE/CISA/NVD, Tier 1: major vendors, Tier 2: journalism, Tier 3: additional CTI), rejects ephemeral URLs (Vertex AI Search, Google Cache)
- `nvdService.ts` — validates CVE severity against free NVD REST API. Rate-limited (5 per 30s), 10s timeout, caps at 10 CVEs per profile

**Data layer:**
- `config/database.ts` — PostgreSQL pool, schema creation (CREATE TABLE IF NOT EXISTS), seeding
- `models/db.ts` — all DB queries (actors, CVEs, sources, trusted URLs/files, chat messages, `getActorByName()` for upsert)
- `data/trustedData.ts` — hardcoded ground-truth data for specific actors (APT29, Volt Typhoon, APT41, etc.) used to override/validate AI output. Includes verified CVEs, aliases, first_seen dates, and sources

### Database Schema (PostgreSQL)

Six tables: `threat_actors`, `actor_cves`, `actor_sources`, `trusted_urls`, `trusted_files`, `chat_messages`. Schema is defined inline in `config/database.ts`. No migration tool — tables are created on startup.

### AI Pipeline Flow

1. User requests actor profile → controller calls `geminiService.generateActorProfile()`
2. Pre-lookup: MITRE ATT&CK STIX data queried for known actor info
3. Trusted context assembled: user-uploaded files + approved URLs fetched from DB
4. **Pass 1 (Research)**: Gemini called with Google Search grounding tool → produces detailed text analysis
5. **Pass 2 (Structuring)**: Second Gemini call extracts research into strictly-typed JSON schema
6. Post-generation validation pipeline:
   - **Step 1**: Override CVEs/aliases/first_seen with trusted CSV ground truth (`trustedData.ts`)
   - **Step 2**: Enrich with MITRE ATT&CK data (aliases, first_seen, ATT&CK URL)
   - **Step 2.5**: Validate CVE severity ratings against NVD
   - **Step 3**: Validate all source URLs (HEAD requests, 5s timeout, tiered whitelist)
   - **Step 4**: Ensure minimum sources exist
7. Result saved to PostgreSQL (upsert by name) and returned to client with `GenerationLog`

### Generation Log

Every profile generation produces a `GenerationLog` (ephemeral, session-only) containing:
- `groundingUrls` — URLs Gemini visited during web search (from grounding metadata)
- `approvedSources` — user-approved URLs, MITRE ATT&CK page, trusted CSV sources
- `trustedFiles` — filenames of uploaded PDF/CSV files used as context
- `steps` — each pipeline step with label, description, and timing

### Shared Types

`shared/types.ts` defines `ThreatActor`, `CVE`, `Source`, `ChatMessage`, `NewsItem`, `ViewState`, `TrustedFile`, `GenerationLogEntry`, `GenerationLog`. Both client and server reference these types (client tsconfig includes `../shared`).

## Important Patterns

- The Gemini model used is `gemini-3-flash-preview` (hardcoded in `geminiService.ts`)
- All Gemini calls use `temperature: 0` for deterministic output
- Server uses ESM modules — TypeScript imports require `.js` extension for local files
- The client has no test framework configured; the server has `validateProfiles.ts` for accuracy testing
- Tailwind CSS is used for styling with a dark/cyber aesthetic (`bg-slate-950`)
- Actor generation uses upsert pattern: `getActorByName()` checks for existing, then creates or updates
- `generateActorProfile()` returns `{ profile, log }` — the controller returns `{ actor, generationLog }`
- Trusted data overrides AI output for known actors — more reliable than prompt engineering
- Alias cross-validation: when passing trusted aliases through `validateAliases()`, they must be included in the `mitreAliases` param so they're treated as "own" names
- MITRE fuzzy matching uses string inclusion (`includes()`), so actors with completely different names won't match — trusted data is the workaround
- Approved Sources panel fetches actor names from `GET /api/sources` on mount (not localStorage)
- Multer file upload limit: 20MB, allowed extensions: .pdf, .csv, .xlsx, .xls
- PDF parsing: `pdfjs-dist` on server side; CSV/XLSX parsing: `xlsx` library

## API Routes

```
/api/actors          GET      — List all actors
/api/actors/:id      GET      — Get actor by ID
/api/actors/generate POST     — Generate new profile (returns { actor, generationLog })
/api/actors/:id/refresh-section POST — Refresh specific section
/api/actors/:id      PUT      — Update actor
/api/actors/:id      DELETE   — Delete actor
/api/chat            POST     — AI chat with context
/api/news            GET      — Live cyber news
/api/sources         GET      — List actor names with trusted sources
/api/sources/:name   GET      — Get URLs and files for an actor
/api/sources/urls    POST     — Add approved URL
/api/sources/urls/:id DELETE  — Remove approved URL
/api/sources/files   POST     — Upload trusted file (multipart)
/api/sources/files/:id DELETE — Remove trusted file
/api/health          GET      — Server health check
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nezeon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Nezeon)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
