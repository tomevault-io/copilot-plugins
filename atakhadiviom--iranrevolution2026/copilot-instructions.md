## iranrevolution2026

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

Iran Memorial is an interactive, open-source memorial website commemorating individuals who lost their lives during the Iranian revolution. It features a Leaflet-based map interface, AI-assisted data extraction from social media, bilingual support (English/Persian), and an admin dashboard for content moderation.

**Key Ethical Consideration**: This is a human rights memorial project. All changes should align with the [Care Protocol](CARE_PROTOCOL.md), which prioritizes victim dignity, family safety, and ethical data handling. AI is used for extraction assistance only—human verification is required before any content goes live.

## Development Commands

```bash
# Local development
npm run dev              # Start Vite dev server

# Build and test
npm run build           # TypeScript + Vite build
npm run typecheck       # TypeScript type checking
npm run lint            # ESLint
npm test                # Run Vitest tests

# Admin/maintenance scripts (require .env file)
npm run backup          # Backup database to JSON
npm run discover        # Auto-discover posts from X/social media
npm run scrape-telegram # Scrape Telegram channel posts
npm run admin-review    # AI-assisted admin review
npm run verify-rtn      # Batch verify RTN (Remember Their Names) sources
```

**Environment Setup**: Copy `.env.example` to `.env` and configure:
- `VITE_OPENROUTER_API_KEY` - Required for AI features (extraction, translation, geocoding)
- `VITE_SUPABASE_URL` + `VITE_SUPABASE_ANON_KEY` - Database connection
- `VITE_SUPABASE_SERVICE_ROLE_KEY` - Required for admin operations (bypasses RLS)

## Architecture

### Dual Data Model

The application supports two data sources:

1. **Supabase (PostgreSQL)** - Primary, with real-time updates
2. **Static JSON** (`public/data/memorials.json`) - Fallback for GitHub Pages

The `dataService.ts` module automatically falls back to static JSON if Supabase is unavailable or fails. When Supabase is configured, it uses real-time subscriptions to auto-refresh the map when data changes.

### Core Modules

- **`src/modules/supabase.ts`** - Supabase client initialization (anon + admin clients)
- **`src/modules/dataService.ts`** - CRUD operations, duplicate detection, merge logic, batch operations
- **`src/modules/ai.ts`** - OpenRouter API integration for:
  - `extractMemorialData()` - Extract victim info from URLs via Jina Reader API
  - `translateMemorialData()` - Bilingual EN/FA translation
  - `geocodeLocation()` / `reverseGeocode()` - Location coordinates
  - `generateText()` - Generic text generation
- **`src/modules/types.ts`** - TypeScript types for `MemorialEntry`, database types
- **`src/modules/map.ts`** - Leaflet map with marker clustering
- **`src/modules/i18n.ts`** - Bilingual translations (English/Persian)
- **`src/main.ts`** - Main app bootstrapping, UI rendering
- **`src/admin.ts`** - Admin dashboard for moderation

### Entry Points

- **`index.html`** → `src/main.ts` - Public memorial map
- **`admin.html`** → `src/admin.ts` - Admin dashboard (requires Supabase auth)

Both are built by Vite as separate entry points (see `vite.config.ts`).

### Data Flow

1. **Public Submission** → `submitMemorial()` creates unverified entry in Supabase
2. **Admin Review** → Admin verifies via dashboard (`verifyMemorial()`) or merges duplicates
3. **Real-time Sync** → Supabase subscription triggers map refresh on changes
4. **Fallback** → If Supabase unavailable, serves static JSON

### Merge Logic

When duplicate entries are detected (same name + city), the system:
1. Merges `source_links` (references) from duplicate into target
2. Deletes the duplicate entry
3. Auto-verifies if source is RTN (Remember Their Names) Telegram channel

See `dataService.ts:findDuplicateMemorial()` and `mergeMemorials()`.

### AI Extraction Pipeline

Scripts in `scripts/` use AI to discover new entries:

1. **Jina Reader API** (`r.jina.ai/http://...`) - Fetches page content as Markdown
2. **OpenRouter AI** - Extracts structured victim data (name, city, date, bio, coords)
3. **Duplicate Check** - Prevents re-submitting known entries
4. **Database Insert** - Submits as unverified for admin review

Key scripts:
- `scripts/discover_x.ts` - Scrape X/Twitter posts for memorial content
- `scripts/batch_verify_rtn.ts` - Auto-verify RTN sources
- `scripts/admin_review_agent.ts` - AI-assisted moderation

## Care Protocol Integration

Per `CARE_PROTOCOL.md`:
- **AI outputs are unverified suggestions** - never auto-publish
- **Avoid extracting sensitive info** (home addresses, relatives' details)
- **Support anonymity** - first name only, initials, or "Anonymous"
- **Social media requires extra care** - prefer reputable sources, don't re-share identifying details
- **Corrections/removals are ethical care** - handle requests quickly

When modifying AI prompts or extraction logic, ensure these safeguards remain.

## Testing

Run individual tests:
```bash
npx vitest run src/modules/__tests__/i18n.test.ts
```

## Deployment

Pushing to `main` triggers GitHub Actions workflow (`.github/workflows/deploy.yml`):
1. Runs `lint` and `typecheck`
2. Builds with Vite (injecting secrets from GitHub Repository Secrets)
3. Deploys to GitHub Pages

**Required GitHub Secrets**:
- `VITE_OPENROUTER_API_KEY`
- `VITE_SUPABASE_URL`
- `VITE_SUPABASE_ANON_KEY`

**Note**: Service role key should NEVER be added to GitHub Actions—admin operations should be done locally.

## Admin Dashboard

Access at `/admin.html` (requires Supabase auth configuration).

Features:
- **Overview** - Stats, AI extraction, batch operations (sync images, translate, geocode)
- **Submissions** - Review pending entries, verify/delete
- **Memorials** - Edit verified entries, merge duplicates
- **Reports** - Handle user-submitted correction requests
- **Editor** - Full CRUD with duplicate detection, AI translation, merge UI

The admin uses the same `dataService.ts` functions but with the service role client to bypass RLS.

## Common Patterns

### Bilingual Fields

All user-facing text has `name`/`name_fa`, `city`/`city_fa`, etc. The `i18n.ts` module determines which to display based on `currentLanguage()`. When adding new fields, add both English and Persian versions.

### Duplicate Detection

Names are normalized (lowercase, trimmed) and compared. The system also handles Persian name matching and recognizes common prefixes (Syed, Mir, etc.). See `main.ts:checkDuplicate()`.

### Sensitive Content

The `sensitive` flag hides bio/testimonials by default. `sensitiveMedia` blurs images/videos. Both show a "Show Content" warning overlay per the care protocol.

### Social Media Embeds

- **X/Twitter**: Uses `twitter-widgets` script (see `src/modules/twitter.ts`)
- **Instagram**: Uses `instagram-embed` script (see `src/modules/instagram.ts`)
- **Telegram**: Native iframe embed with `?embed=1` parameter

---
> Source: [atakhadiviom/IranRevolution2026](https://github.com/atakhadiviom/IranRevolution2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
