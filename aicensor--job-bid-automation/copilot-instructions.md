## job-bid-automation

> AI-powered resume tailoring engine that automatically optimizes resumes for specific job postings to achieve top ATS scores. Built with Next.js 15, TypeScript, SQLite, and multi-provider LLM integration.

# CLAUDE.md — Job Bid Automation Project Guide

## Project Overview

AI-powered resume tailoring engine that automatically optimizes resumes for specific job postings to achieve top ATS scores. Built with Next.js 15, TypeScript, SQLite, and multi-provider LLM integration.

**Live URL:** http://156.67.24.31:3000 (Ubuntu VPS, Contabo)
**Dev URL:** http://localhost:3000
**Default admin:** `admin` / `admin123` (or `ADMIN_DEFAULT_PASSWORD` env var)

## Architecture

```
User Flow:
  Admin  → Upload resumes → Assign to bidmen (with main skills, instructions, strict mode)
  Bidman → Paste job URL → Auto skill-match → Select resume → Tailor → Download PDF

Pipeline Flow:
  1. Parse JD → structured requirements + keywords
  2. Score base resume (before)
  3. Analyze gaps (missing skills, achievements)
  4. Rewrite bullets (LLM, STAR format) + Generate summary (parallel)
     - additionalInstructions injected from admin's per-resume config
  5. Reorder sections by relevance
  6. 3-pass refinement:
     a. Inject missing keywords
     b. Clean AI phrases
     c. Truth validation (skipped when strict_truth_check=OFF)
  7. Score tailored resume (after)
  8. Loop if score < threshold (default 85, max iterations configurable)
```

## Tech Stack

- **Framework:** Next.js 15 (App Router), React 19, TypeScript
- **Database:** SQLite (better-sqlite3) at `data/app.db`
- **Auth:** HMAC-SHA256 signed session cookies (7-day TTL), native `bcrypt` (8 rounds)
- **AI:** NVIDIA NIM (free, primary) → Gemini (free, fallback) → OpenRouter (paid, last resort)
- **AI SDK:** Vercel AI SDK v4 (`ai@4.3.19`), `@ai-sdk/google@1.2.22` (v1 for SDK v4 compat)
- **PDF:** Puppeteer for generation, pdf-parse for extraction
- **DOCX:** `docx` library for export, `mammoth` for import
- **Styling:** Tailwind CSS v4
- **Validation:** Zod schemas for LLM output parsing

## Directory Structure

```
src/
  app/                  # Next.js App Router
    (auth)/             # Login, Register (no sidebar)
    (dashboard)/        # Shared tools: Tailor, History, Jobs, Settings
                        # Layout is auth-aware — shows Admin links for admin users
    admin/              # Admin pages: Users, Resume assignments
    bidman/             # Bidman pages: Bid workflow, Log
    api/
      auth/             # login, logout, register
      admin/            # users CRUD, assignments, user password reset
      bidman/           # tailor, log, status, fit-check, job-history
      jobs/             # search, scrape, parse, [id]
      queue/            # CRUD, run (SSE), auto
      resume/           # list, upload
      tailor/           # main tailoring endpoint
      score/            # scoring endpoint
      export/           # DOCX/PDF export
      history/          # tailoring results
      settings/         # app settings
    page.tsx            # Root redirect: admin→/admin, bidman→/bidman, else→/login
  core/                 # Core AI/ML logic
    parser/             # Resume (PDF/DOCX→JSON) + JD parsing
    tailor/             # Pipeline, gap analysis, rewriter, summary generator
    scorer/             # ATS, semantic, seniority, readability, composite
    refiner/            # Keyword injection, AI cleanup, truth validation
    scraper/            # LinkedIn search, universal job scraper, browser resolver
    validators/         # Blocked fields, quality gates
  ai/                   # Provider config, timeouts, prompts (markdown files)
  integrations/         # DOCX, PDF, Google Sheets, LinkedIn bot
  lib/                  # DB, auth, types, queue, logging, format
  components/           # React components by domain
data/
  app.db                # SQLite database (auto-created on first request)
  base-resume/          # Uploaded base resumes (PDF/DOCX)
  output/               # Generated tailored resumes (JSON)
  achievement-bank.yaml # Real achievements in STAR format
  preferences.yaml      # Tailoring preferences (tone, emphasis, seniority)
  bidman-log.json       # Bidman activity log (also synced to Google Sheets)
  queue.json            # Job queue state
```

## Database Schema (SQLite)

```sql
users (id, username, password_hash, role[admin|bidman], status[pending|approved|disabled], created_at, updated_at)
sessions (id, user_id FK, created_at, expires_at)
resume_assignments (id, user_id FK, resume_filename, main_skills, tailoring_instructions, strict_truth_check[0|1], created_at, UNIQUE(user_id, resume_filename))
job_search_history (id, user_id FK, job_url, job_title, company, tech_stacks, industry, job_data[JSON], created_at)
```

## Key Files to Know

| File | Purpose |
|------|---------|
| `src/core/tailor/pipeline.ts` | Main tailoring orchestrator — the heart of the system |
| `src/core/tailor/rewriter.ts` | LLM bullet rewriting with achievement bank + additional instructions |
| `src/core/tailor/summary-generator.ts` | LLM summary generation + additional instructions |
| `src/core/refiner/truth-validator.ts` | Prevents fabricated content (skipped when strict=OFF) |
| `src/ai/providers.ts` | Model registry + fallback chain config |
| `src/ai/timeout.ts` | 60s timeout per LLM call + model fallback |
| `src/lib/db.ts` | SQLite schema, seed, types. Delete `data/app.db` after schema changes |
| `src/lib/auth.ts` | Session management (HMAC signed cookies, native bcrypt) |
| `src/middleware.ts` | Route protection (Edge Runtime, Web Crypto API — not Node crypto) |
| `src/app/api/bidman/tailor/route.ts` | Bidman tailoring API — threads instructions + strictTruthCheck |
| `src/components/bidman/BidmanWorkflow.tsx` | Bidman UI workflow + client-side skill-match fit check |
| `src/components/bidman/JobUrlInput.tsx` | URL input + DB-backed job search history |
| `src/components/admin/ResumeManager.tsx` | Resume upload + assignment (main skills, instructions, strict toggle) |
| `data/achievement-bank.yaml` | Achievement bank for bullet enrichment |
| `data/preferences.yaml` | Tailoring preferences (tone, emphasis, seniority) |

## AI Provider Chain

```
Task        Primary (free)           Fallback1 (free)      Fallback2 (paid)
rewrite     NVIDIA NIM qwen3-80b     Gemini 2.5 Flash      OpenRouter Claude Sonnet
parse       NVIDIA NIM qwen3-80b     Gemini 2.5 Flash      OpenRouter Claude Sonnet
analyze     NVIDIA NIM qwen3-80b     Gemini 2.5 Flash      OpenRouter GPT-4o-mini
embedding   NVIDIA NIM nv-embedqa    —                      OpenRouter text-embedding-3-small
```

Timeout: 60s per call. On timeout/error, automatically tries next model in chain.

## Scoring System (0-100)

| Dimension | Weight | Source |
|-----------|--------|--------|
| ATS Keywords | 35% | TF-IDF + exact + synonym matching |
| Semantic Similarity | 20% | Embedding cosine similarity |
| Seniority Signals | 20% | Action verbs, metrics, leadership |
| Readability | 10% | Structure, bullet clarity, formatting |
| Achievement Quality | 15% | STAR format, quantification |

Threshold: 85/100 default. Typical improvement: +20-40 points.

## Resume-Job Fit Check (Bidman)

Two-stage check after job is scraped. Each stage shows a badge in the resume selector.

**Stage 1 — Skill Match** (instant, client-side in `BidmanWorkflow.tsx`):
- Checks each resume main skill against job title, primary tech (top 3), and secondary tech
- Score = average points per resume skill: title match 100, primary 70, secondary 30, no match 0
- Badge colors: Green >= 70% | Yellow >= 30% | Red < 30% (`STAGE1_THRESHOLD`)

**Stage 2 — AI Score** (async LLM via `/api/bidman/fit-check`):
- Runs only for resumes that pass Stage 1 (>= 30%)
- LLM scores mainSkills vs full job context (title + description + tech stacks)
- Badge colors: Green >= 70% | Yellow >= 40% (`LLM_THRESHOLD`) | Red < 40%

**Behavior:**
- Auto-selects best matching resume after each stage
- If ALL resumes below 30%: warning with "Proceed Anyway" or "Try Different Job"
- Main skills displayed as tags under each resume name
- Missing skills shown below red badges

This is intentionally separate from the full composite scorer — it's a fast gate, not a precise ATS prediction.

## Additional Instructions Flow

Admin sets per-resume `tailoring_instructions` and `strict_truth_check` in `/admin/resumes`.

```
DB (resume_assignments) → bidman page → BidmanWorkflow → TailorAction → POST /api/bidman/tailor
  → tailorResume({ additionalInstructions, strictTruthCheck })
    → rewriteBullets() — injected as "## Additional Instructions" in LLM prompt
    → generateSummary() — injected as "## Additional Instructions" in LLM prompt
    → pipeline step 7c — truth validation skipped if strictTruthCheck=false
```

Use cases: achievement expression style, resume writing tone, adding specific content/technologies.

## Auth & Roles

- **Admin:** Full access — user management, resume uploads/assignments, all tools (Tailor, History, Find Jobs, Job Queue, Settings)
- **Bidman:** Bid workflow (paste URL → tailor → PDF), own log only, assigned resumes only, job search history
- **Middleware:** Edge Runtime with Web Crypto HMAC verification (not Node.js crypto)
- **Token format:** `sessionId|expiresAt|hmacSignature` (pipe-delimited — ISO dates have colons which break `:` delimiter)
- **Password hashing:** Native `bcrypt` with 8 rounds (~37ms). Switched from `bcryptjs` (pure JS, ~244ms) for performance.
- **Stale cookie handling:** If session exists in cookie but not in DB (after DB reset), root page redirects through `/api/auth/logout?redirect=/login` to clear cookie and prevent redirect loops.

## Route Groups & Layouts

| Group | Sidebar | Auth | Notes |
|-------|---------|------|-------|
| `(auth)` | None | Public | Login, Register pages |
| `(dashboard)` | Full sidebar (admin-aware) | Required | Tailor, History, Jobs, Queue, Settings |
| `/admin` | Admin sidebar (identical links) | Admin only | Users, Resume Assign |
| `/bidman` | Bidman sidebar (Bid, Log) | Bidman only | Bid workflow, Tailor Log |

The `(dashboard)` and `/admin` sidebars are kept identical (Admin, Resume, Job Search, System sections). The `(dashboard)` layout hides the Admin section for non-admin users.

## Common Commands

```bash
npm run dev              # Dev server (hot reload, slow)
npm run build            # Production build
npx next start -p 3000   # Production server (fast)
npx tsc --noEmit         # Type-check
sudo -u tailor pm2 stop tailor   # Stop PM2 production process (uses port 3000)
sudo -u tailor pm2 start tailor  # Restart PM2 production process
```

## Deployment (Production)

```bash
npm run build
npx next start --port 3000       # or use PM2:
sudo -u tailor pm2 restart tailor
```

PM2 process runs as user `tailor` on port 3000. Stop it before running `next dev`.

## Environment Variables

See `.env.example` for full list. Key ones:
- `NVIDIA_API_KEY` — Primary LLM provider (free)
- `GOOGLE_GEMINI_API_KEY` — Fallback LLM (free)
- `OPENROUTER_API_KEY` — Paid fallback
- `GOOGLE_SERVICE_ACCOUNT_KEY_PATH` — Google Sheets service account JSON file path
- `GOOGLE_SHEET_ID` — Target spreadsheet ID
- `GOOGLE_SHEET_NAME` — Sheet tab name (default: `Sheet1`)
- `SESSION_SECRET` — HMAC signing key for auth tokens
- `ADMIN_DEFAULT_PASSWORD` — Admin password on first seed (default: `admin123`)

### Changing Google Sheets Account

1. Create new service account in Google Cloud Console → download JSON key
2. Share the Google Sheet with the service account email (Editor access)
3. Update `.env`:
   ```
   GOOGLE_SERVICE_ACCOUNT_KEY_PATH=./new-key-file.json
   GOOGLE_SHEET_ID=new-sheet-id
   ```

## Known Issues & Constraints

1. **LLM timeout:** NVIDIA NIM sometimes exceeds 60s on large resumes → falls back to Gemini. Do not increase timeout beyond 60s.
2. **Skills section not tailored:** The rewriter only modifies experience bullets + summary, not the skills section. Skills remain from base resume even when mismatched.
3. **Job titles can be fabricated:** The rewriter can change job titles (e.g., "Software Engineer" → "Python DevOps Engineer"). Truth validator checks company/dates but not titles aggressively enough.
4. **~~Keyword stuffing~~** *(mitigated 2026-04-05)*: Rewriter prompt now limits bold to 2 tech terms per bullet, AI cleaner catches over-bolding and repeated bold keywords.
5. **Clipboard on HTTP:** `navigator.clipboard` doesn't work on plain HTTP. All copy buttons use `execCommand('copy')` fallback.
6. **DB schema changes:** `CREATE TABLE IF NOT EXISTS` won't add new columns. Must delete `data/app.db` and restart after schema changes. This wipes all users — re-create them manually or via the register flow.

## Planned Features

- [x] Pre-tailoring resume-job fit check (client-side skill matching)
- [x] Admin per-resume: main skills, tailoring instructions, strict truth toggle
- [x] Job search history (DB-backed, per bidman)
- [x] Job search results caching (sessionStorage, survives tab switches)
- [ ] Skills section tailoring — reorder/filter skills to match JD requirements
- [ ] Middle-ground truth checking — allow new content but flag for review
- [ ] Bidman activity dashboard — stats per bidman (bids/day, avg score improvement)
- [ ] Resume versioning — track changes across tailoring runs
- [ ] Job title fabrication guard in truth validator

## Changelog

### 2026-04-05 — Anti-AI Detection & Fit Score Fix

**Problem:** Tailored resumes were flagged by AI detectors due to:
- 11/27 bullets had fabricated "by XX%" metrics not in the original resume
- Same bold keywords repeated 3-4x across bullets (`**responsive UI**` 4x, `**SQL-based database schemas**` 3x)
- Every bullet followed identical structure: "Verb + **bold keyword** + context, reducing X by Y%"
- Soft skills bolded (`**code reviews**`, `**cross-functional collaboration**`)

**Changes:**
1. **Rewriter prompt** (`src/ai/prompts/rewrite-bullets.md`) — complete rewrite
   - Metrics rule: only include numbers from the original bullet; no number in source = no number in output
   - Bold limit: max 2 tech terms per bullet, never bold soft skills, never repeat same bold across resume
   - Structure variation: must mix result-first, action-first, context-first; not every bullet needs a metric
   - Added explicit "What Makes It Detectable as AI" section listing exact patterns to avoid

2. **Summary prompt** (`src/ai/prompts/generate-summary.md`) — rewritten for plain human voice
   - No fabricated metrics, no marketing speak, specific examples over generic claims

3. **AI phrase cleaner** (`src/core/refiner/ai-phrase-cleaner.ts`) — expanded detection
   - New structural checks: fabricated metric patterns, over-bolding (3+ per bullet), soft-skill bolding
   - Cross-bullet analysis: flags bullets sharing overused bold keywords (3+ occurrences)
   - Feeds specific detected issues to the cleanup LLM for targeted fixes

4. **Keyword injector** (`src/core/refiner/keyword-injector.ts`) — dialed back
   - No adding bold, no fabricated metrics, minimal edits only, skip if unnatural

### 2026-04-05 — Stage 1 Fit Score Algorithm Rewrite

**Problem:** Jobs with 15+ tech stacks diluted the score. A C#/.NET resume scored 5% for a DevOps job listing `.NET` as one of 15 skills — the denominator was the job's total weighted skills, so 2-3 main skills could never score well.

**Fix:** Flipped the perspective from "how much of the job does this resume cover" to "are the resume's main skills what this job primarily wants":
- For each resume main skill, check WHERE it appears in the job
- Title match → 100 pts (job is fundamentally about this skill)
- Primary tech (top 3) → 70 pts (core requirement)
- Secondary tech (rest) → 30 pts (listed but not the focus)
- No match → 0 pts
- Score = average across resume's main skills

**Result:** C#/.NET resume now scores 30% for DevOps job (skills present but secondary) vs 70% for a .NET Developer job (skills are primary). Previously both scored ~5%.

## Development Gotchas

- **Always delete `data/app.db`** after schema changes — then hit any endpoint to re-seed admin
- **Middleware runs in Edge Runtime** — cannot use Node.js `crypto`, must use Web Crypto API
- **Session token delimiter is `|`** not `:` — ISO dates contain colons which broke the original parser
- **`@ai-sdk/google` must be v1.x** for compatibility with `ai@4.x` — v3.x requires AI SDK v5
- **`bcrypt` (native)** not `bcryptjs` (pure JS) — 6.5x faster, but requires native compilation
- **Production build required** for acceptable performance — `next dev` is very slow on VPS
- **PM2 auto-restarts** the production process as user `tailor` — must `pm2 stop tailor` before dev
- **Root `/` page** is a redirect-only page (admin→/admin, bidman→/bidman) — no dashboard at root
- **Fit check is client-side** — uses `main_skills` from DB, matched against `mainTechStacks` from scraper. No LLM involved. Admin must set main skills per resume for it to work.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aicensor)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/aicensor)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
