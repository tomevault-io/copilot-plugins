## prospectiq

> LeadScope AI — local-business lead intelligence platform.

# ProspectIQ — Claude Code Instructions

## Project overview

LeadScope AI — local-business lead intelligence platform.
Stack: FastAPI + SQLAlchemy + MariaDB (`apps/api`) / React + TypeScript + Vite (`apps/web`).
Pipeline: search-job → LeadDiscoveryOrchestrator (BackgroundTasks) → normalize → score → outreach draft.
Roles: admin, agency_manager, sales_user.

## Active goal — /goal

When the user runs /goal, do the following autonomously without asking for explanation:

### Step 1 — Read these files first (no code yet)

- apps/web/src/app/router.tsx
- apps/web/src/ (all search and lead route components)
- apps/api/app/modules/search_jobs/api.py
- apps/api/app/workers/orchestration/lead_discovery.py
- apps/api/app/modules/leads/ (all files)
- apps/api/app/modules/outreach/ (all files)

### Step 2 — Identify and fix: Search UX

- Collapse multi-screen search creation into a single drawer: business type → location → filters → launch
- Add Zod + React Hook Form validation to the search form
- Add empty states to: search history list, leads table, map view
- Add loading skeletons (no spinners alone)
- Add real-time job progress via SSE: GET /api/v1/search-jobs/{job_id}/stream
  - Events: { stage, progress: 0-100, message }
  - Frontend: useJobStream(jobId) hook with EventSource + exponential back-off reconnect
  - Replace static "Running..." badge with live progress bar + stage label

### Step 3 — Identify and fix: Outreach

- Add "Copy draft" button with clipboard toast
- Add send/schedule stub endpoint: POST /api/v1/outreach/{id}/send (log only, no real SMTP yet)
- Track outreach status per lead: draft → sent → replied (add DB column if needed)
- Show outreach status badge on leads table row

### Step 4 — Lead table polish

- Column visibility toggle (score, web_presence, phone, category) — persist in localStorage
- Quick-filter bar: score range slider + has_website toggle + has_phone toggle
- Scoring breakdown as bar chart on lead detail page (not just a number)

## Constraints (always apply)

- No schema changes in Steps 1 or 4 — schema only in Steps 2/3 if required
- All new API endpoints under /api/v1/ with existing auth middleware
- TypeScript only — zero `any` types
- Run `ruff check` + `mypy` before finishing any Python file
- Run `npm run build` — zero TS errors before finishing any frontend file
- Write CHANGES.md entry per step

## Output format per step

1. List files read
2. Bullet plan (max 10 items)
3. Implement — no approval needed, go autonomous
4. Print diff summary + lint results

---
> Source: [YusufJojeh/ProspectIQ](https://github.com/YusufJojeh/ProspectIQ) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
