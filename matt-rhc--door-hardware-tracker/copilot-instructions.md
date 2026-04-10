## door-hardware-tracker

> A Next.js SaaS application that parses door hardware submittal PDFs and turns them into a trackable, QR-code-enabled checklist for construction teams.

@AGENTS.md

# Door Hardware Tracker

A Next.js SaaS application that parses door hardware submittal PDFs and turns them into a trackable, QR-code-enabled checklist for construction teams.

**Live:** trackdoorhardware.app
**Repo:** github.com/matt-RHC/door-hardware-tracker

## Tech Stack

- **Framework:** Next.js 16.2 (App Router) + TypeScript, deployed on Vercel Pro
- **Build:** Turbopack (dev + production) — stricter than standard tsc, see rules below
- **Database:** Supabase (Postgres + Auth + RLS + Realtime)
- **AI:** Anthropic API (claude-sonnet-4-20250514) for PDF parsing + review
- **PDF extraction:** Python pdfplumber serverless function (`/api/extract-tables.py`)
- **Hosting:** Vercel Pro plan, Fluid Compute enabled, 300s maxDuration on PDF/sync routes

## Critical Rules

### 1. Check Before You Build
**HARD RULE.** Before creating any new file, table, endpoint, or component, search `origin/main` and the full codebase for existing implementations. This repo evolves across many sessions — the feature may already exist under a different name. If a "small fix" turns into building new infrastructure, STOP and tell the user: "This is bigger than expected — here's what I'm seeing" before proceeding. Default to the smallest change that solves the problem.

### 2. No Plans Without Merged Code
**HARD RULE.** Do not write new plans, research docs, or architecture documents until the code from the last plan is merged to `main` and tested. If a session only produces documents and no code changes, that is a problem. One bug fix at a time: fix → test → merge → next.

### 3. Milestones Over Speed
Complete and test one feature end-to-end before starting the next. No placeholder/TODO code in production. Correctness over speed, always.

### 4. Accuracy Over Automation
Auto-detect where possible, but if uncertain, **ask the user** rather than guessing. Being interactive is fine. A few extra clicks is acceptable if it prevents bad data. We are NOT pushing AI slop.

## Turbopack TypeScript Rules

Turbopack's type checker in Next.js 16.2 is significantly stricter than `tsc --noEmit`. These patterns caused 6 consecutive failed Vercel deploys before we learned them:

1. **Truthy `&&` guards do NOT narrow nullable types.** `if (x && x.prop)` still fails → use `x?.prop`
2. **`?.` on left of `&&` does NOT carry narrowing to right.** `x?.chunks && x.chunks.length` fails → use `(x?.chunks?.length ?? 0) > 0`
3. **Spread on optional arrays fails inside `if` guards.** `if (x.arr) push(...x.arr)` fails → use `push(...(x.arr ?? []))`
4. **`!== null` guards are NOT sufficient.** Must use `?? undefined` or `!` assertion after compound checks.

**Always use `?.`, `??`, and `?? []` for nullable access. Never rely on `if` guards or `&&` for narrowing.**

## Git Workflow

Git operations run directly in the working directory — no `/tmp/` clone needed.

1. Set git identity before first commit: `git config user.email "matt@rabbitholeconsultants.com" && git config user.name "Matthew Feagin"`
2. Default branch is `main`. There is no `master` branch.
3. Use PRs for all changes — enable **"Automatically delete head branches"** in GitHub repo settings to prevent branch buildup.

## Database Schema (Key Tables)

| Table | Purpose |
|---|---|
| `projects` | Job info, Smartsheet IDs, `last_pdf_hash` for dedup |
| `openings` | Door records: door_number, hw_set, location, fire_rating, hand |
| `hardware_items` | Per-opening items: name, qty, model, finish, manufacturer, sort_order, install_type |
| `checklist_progress` | Multi-step workflow: received → pre_install (bench) / installed (field) → qa_qc |
| `attachments` | Per-opening files: floor plans, door/frame drawings |
| `project_members` | RLS enforcement — all tables require project membership |
| `smartsheet_row_map` | Sync state for Smartsheet integration |

**Key decisions:**
- Per-opening quantities (hinges=3-4, closers=1), NOT project totals
- Door and Frame added as checkable items per opening (pair detection for double doors)
- Project creation uses admin Supabase client to bypass RLS chicken-and-egg problem

## PDF Parsing Architecture

Every door hardware submittal PDF has three layers:

**Layer 1 — Opening List (Master Table):** Clean tabular grid. Columns: Opening | Hdw Set | Hdw Heading | etc. This is the **single source of truth** for all door-to-set assignments. Machine-readable, minimal LLM needed.

**Layer 2 — Hardware Schedule (Set Definitions):** One page per hardware set. Header with set ID and assigned doors. Body with item list (qty, name, model, finish, manufacturer).

**Layer 3 — Reference Tables:** Manufacturer codes, finish codes, option codes. Static lookups.

### 3-Phase Extraction Strategy

1. **Phase 1:** Extract the Opening List via direct table extraction (pdfplumber). Output: complete door roster.
2. **Phase 2:** Extract Hardware Set definitions. Parse one representative set thoroughly, use as template for similar sets. Cross-validate against Phase 1.
3. **Phase 3:** Extract reference tables. Decode abbreviations. Final validation: flag orphaned doors/sets.

### Pipeline Flow
```
classify-pages.py → detect-mapping.py → ColumnMapperWizard (user)
  → Punchy Checkpoint 1: Column Mapping Review
  → extract-tables.py (pdfplumber)
  → Punchy Checkpoint 2: Post-Extraction Review
  → quantity normalization
  → Punchy Checkpoint 3: Quantity Sanity Check
  → merge → ImportReviewTable (user review) → save
```

- PDFs ≤ 45 pages: full pipeline, no chunking
- PDFs > 45 pages: client-side splitting into 35-page chunks via pdf-lib
- Resubmissions use the compare wizard for revision tracking
- SHA-256 hash prevents duplicate uploads

### Punchy AI Review Layer (S-067)

**Punchy** is the AI persona for the extraction pipeline — a senior DFH consultant with 25 years in commercial construction. Punchy reviews extraction results at 3 checkpoints and provides confidence-rated observations.

**Checkpoints:**
1. **Column Mapping Review** — checks if expected fields (location, fire_rating, hand) are unmapped and suggests where to find them in the PDF
2. **Post-Extraction Review** — finds missing doors/sets, wrong assignments, field splitting issues. If pdfplumber found very few doors (schedule format), Punchy does heavier lifting extracting inline door assignments
3. **Quantity Sanity Check** — validates normalized quantities against DFH standards (hinge count, fire-rated hardware, pair door rules, IBC egress requirements)

**Key files:**
- `src/lib/punchy-prompts.ts` — shared system prompts for all checkpoints
- `src/app/api/parse-pdf/chunk/route.ts` — 3 Punchy checkpoints wired into chunk processing
- `src/app/api/parse-pdf/route.ts` — same checkpoints for non-chunked flow
- `src/lib/types/index.ts` — `PunchyObservation`, `PunchyCorrections`, `PunchyColumnReview`, `PunchyQuantityCheck` types

**Model:** `claude-haiku-4-5-20251001` (cost-efficient, production-proven)
**Confidence scoring:** Every observation rated high/medium/low so users know what to verify

## UI / UX Notes

- **Theme:** Borderlands industrial — cel-shaded, glow-cards, corner-brackets, Orbitron font, cyan accent (#5ac8fa)
- **Progress loader:** Status messages must be short and ambient (under ~40 chars). Never dump raw data (set IDs, door numbers) into the visual. Detailed info goes in console or collapsible details.
- **Apply-to-all pattern:** When user edits a hardware item, check if matching items exist across other openings. Offer "Apply to all N" / "Just this one" modal.

## What Works (as of S-031)

- PDF upload → column mapping → extraction → pattern validation → user review → DB insert (end-to-end)
- Revision flow: re-upload PDF → compare → apply changes
- Bench/field classification with apply-to-all
- Edit items with apply-to-all for matching items across openings
- Pattern consensus validation with flagged door review
- Column mapper with cover-page false-positive prevention
- Smartsheet one-way sync (manual + daily cron at 6am UTC)
- Inline editing, tabbed door cards, QR codes, CSV export, auth, advanced filters

## Testing

### Golden PDF Test Suite
- **15 training PDFs** in `test-pdfs/training/`: 5 grid-format, 8 schedule-format, 1 kinship, 1 mixed
- **Test fixtures** symlinked in `tests/fixtures/` (e.g., `SMALL_081113.pdf` → `../../test-pdfs/training/grid-MCN.pdf`)
- **Baselines** in `tests/baselines/` — JSON snapshots of expected pipeline output per PDF
- **Ground truth** in `tests/ground-truth/` — human-verified expected door/set counts per PDF
- Python endpoint health check: `/api/extract-tables.py` has text-layer detection
- Known limitation: pdfplumber does not work on image-based/scanned PDFs

### Two-Layer Test Strategy
1. **Baseline tests** (`tests/test_baselines.py`): pdfplumber-only, no API calls. Validates door counts, set counts, set IDs, item quantities. Run with `python -m pytest tests/test_baselines.py -v`. Currently 51 tests across 13 PDFs.
2. **Punchy AI review tests** (`tests/test_pipeline_funnel.py`): Sends extraction results through Punchy (Claude Haiku) for domain-expert review. Requires `ANTHROPIC_API_KEY` env var and `--run-ai-review` flag. Run with `python -m pytest tests/test_pipeline_funnel.py -v --run-ai-review`.

### Mandatory Test Protocol
- **Any pipeline change** must be tested against ALL 13 golden PDFs (not just the 3 original ones)
- **After running tests**, log results to Smartsheet Metrics Log (2206493777547140) with: session ID, PDF name, expected vs extracted doors/sets, accuracy %, pipeline duration, build commit, and notes
- **Regenerating baselines**: Run `python tests/create_expected.py <PDF_NAME>` for each fixture, then copy to the `-baseline.json` naming convention the tests expect
- **When counts change**, update both `tests/baselines/` and `tests/ground-truth/` files, and update any hardcoded assertions (e.g., `test_107_doors`)

### Metrics Tracking & Trend Analysis
**HARD RULE.** Every test run must be logged to the Smartsheet Metrics Log (2206493777547140). This enables:
- **Trend detection**: Track whether door/set accuracy improves or regresses across sessions
- **Regression alerts**: Compare current run against previous session's metrics — flag any accuracy drops
- **Positive signal tracking**: Note which code changes produced accuracy improvements and why
- **Schedule-format progress**: Track the gap between expected=0 and extracted=N for schedule PDFs as the inline parser improves
- **Punchy effectiveness**: When AI review tests run, note whether Punchy caught issues that pdfplumber missed

At session start, read the last 2-3 runs from the Metrics Log to understand the current accuracy baseline. At session end, compare your test results against those baselines and call out trends in the session summary. If any PDF's accuracy dropped, flag it immediately — do not wait for session end.

## Session Protocol

Every session must follow this protocol. These rules are non-negotiable and should not require user reminders.

### Session Start
1. **Read Smartsheet.** Check project plan (4722023373688708), session log (1895373728599940), and metrics log (2206493777547140) for current state before doing anything.
2. **Confirm boilerplate.** First response must list all tracked rules from this file so the user can verify they're loaded. Quick numbered list, no elaboration needed.
3. **Check for unmerged work.** If there's an active plan or open PR from the last session, work on THAT first.

### During Session
4. **Track extraction metrics.** Any pipeline change must be tested against ALL 13 golden PDFs and logged to the metrics sheet (2206493777547140). See "Testing > Metrics Tracking & Trend Analysis" for full protocol.
5. **Compare against prior metrics.** Before logging new test results, read the most recent metrics run and compare. Flag accuracy regressions immediately. Note positive trends in session summary.
6. **Datestamp everything.** Memory files, session entries, project plan updates, and commit messages include dates (YYYY-MM-DD).
7. **Preserve data.** Never delete from memory files, docs, or Smartsheet rows without asking. Append and datestamp instead of overwriting.

### Session End
8. **Update Smartsheet session log.** Add or update session row with: topics, decisions, tasks completed, pending items, status.
9. **Update project plan.** Mark resolved items Done with date. Add new findings as Open rows.
10. **Update memory.** If anything was learned about the user, project, or workflow that future sessions need, save it.
11. **Summarize test trends.** If tests were run this session, include a brief trend comparison vs. the prior metrics run in the session summary (e.g., "MEDIUM door accuracy stable at 100% since S-072; sched-DT FPs reduced from 116→112").

## Output Transparency (MANDATORY)

Every response, commit message, and session summary **must** end with a clear status block. No ambiguity about what happened.

### Format:
```
DONE:
- [list every change actually made and verified]

PLANNED (not yet done):
- [list anything discussed/designed but NOT yet in code]

NOTICED BUT DID NOT TOUCH:
- [list any issues, bugs, or concerns spotted during this work that were outside scope]
- [if none, say "None"]

SHADOW CHANGES:
- [list ANY change made that was NOT explicitly requested — even minor refactors, lint fixes, import reorders, comment edits]
- [if none, say "None"]
```

### Rules:
- **Never describe planned work in a way that sounds like it was completed.** Use future tense ("will need to") for plans, past tense ("added", "fixed") only for done work.
- **Never silently fix adjacent issues.** If you see a bug near the code you're working on, report it under "Noticed but did not touch" — do NOT fix it without asking first.
- **If you made zero code changes, say so explicitly.** "No files were modified in this session" is a valid and important status.
- **Commit messages must reflect only what the commit contains.** Do not reference planned follow-up work in commit messages.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/matt-RHC) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
