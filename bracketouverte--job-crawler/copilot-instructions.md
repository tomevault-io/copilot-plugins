## job-crawler

> Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

# CLAUDE.md — job-scrapper

Project root: `/home/$USER/www/job-scrapper`

> Last updated: 2026-05-10
> Update this file when: model status changes, new API endpoints, new hard rules, pipeline prompt changes.

---

## What this project does

Job scraper + fit analyzer + local viewer. It crawls job boards via ATS provider APIs, scores listings against a candidate profile using LLMs, and surfaces results through a local web UI with filtering, analysis pipelines, and a Discord notification system.

---

## Architecture overview

```
job-scrapper/
├── crawler/          # Provider-specific scrapers
├── matcher/          # Python — LLM scoring pipelines
│   ├── job_post_parser.py
│   ├── job_fit_analyzer.py     # Quick / Maverick pipeline
│   ├── ensemble_runner.py      # Full / Ensemble pipeline
│   ├── benchmark.py
│   ├── benchmark_ensemble.py
│   └── career-ops/             # User-owned profile files — NEVER TOUCH
│       ├── _profile.md
│       ├── profile.yml
│       ├── cv.md
│       └── portals.yml
└── viewer/           # Node/Express + TypeScript
    ├── src/
    │   ├── server.ts           # Express API + SQLite
    │   └── lib/
    │       ├── types.ts
    │       ├── queue.ts
    │       └── config.ts
    └── public/
        ├── index.html          # Active UI (index.old.html = retired Bootstrap UI)
        ├── feed.js             # All UI logic
        ├── feed.css            # All styles
        └── saved-searches.json # User-editable search presets
```

---

## Docker policy — MANDATORY

**Never run npm / npx / python directly. Always use Docker.**

| Change type | Command |
|---|---|
| `server.ts` or any TypeScript | `docker compose build viewer && docker compose up -d viewer` |
| Static files (`feed.js`, `feed.css`, `index.html`) | `docker compose build viewer && docker compose up -d viewer` |
| Python matcher changes | `docker compose build matcher` |

Rebuild automatically when files change — do not ask for permission first.

---

## Environment variables (docker-compose.yml)

| Var | Purpose |
|---|---|
| `NVIDIA_API_KEY` | Required for all LLM calls |
| `NVIDIA_MODEL` | Default scorer — overrides Python default |
| `NVIDIA_ENSEMBLE_SCORERS` | Comma-separated scorer model list |
| `CATALOG_DB` | `/app/state/catalog.sqlite` |
| `CAREER_OPS_DIR` | Profile directory inside matcher, default `career-ops` |
| `SCORE_NOTIFY_MIN_SCORE` | Min score threshold for Discord notifications and To-Apply bucket |

---

## API endpoints (server.ts)

| Method | Path | Notes |
|---|---|---|
| GET | `/api/jobs` | Paginated job list. Filters: `title`, `location`, `company`, `sources`, `days`, `page`. `days` filters by `first_seen_at` |
| GET | `/api/job` | Single job + analysis. Params: `provider`, `source_key`, `job_id` |
| GET | `/api/job-parsed` | Live parse via `job_post_parser`. Query params only — Workday `source_key` contains `/` |
| POST | `/api/match-runs` | Start analysis run. Body: `{job_keys, mode}` where mode = `"maverick"` or `"ensemble"` |
| GET | `/api/match-runs/:id` | Poll run status |
| GET | `/api/stats` | Job counts by provider |
| GET | `/api/config` | Ensemble model config |
| GET | `/api/sources` | Available sources |
| GET | `/api/queue` | Queue items |
| POST | `/api/queue/:id/retry` | Retry a failed queue item |

---

## Crawler — supported providers

Greenhouse, Lever, BambooHR, Ashby, Workday, TeamTailor, Workable, SmartRecruiters, iCIMS.
HiBob: SPA, not implemented.

### iCIMS specifics
- Mechanism: XML sitemap (`https://careers-{slug}.icims.com/sitemap.xml`)
- Fields available: title (from URL slug), job_url, job_id
- Fields NOT available: location, employment_type, compensation, posted_at (sitemap only)
- Recommended concurrency: 5 (safe), 8–10 (balanced/aggressive)
- Source file: `sources/icims.json` — 9,937 slugs

---

## Matcher — scoring pipelines

### Quick pipeline (Maverick)
File: `job_fit_analyzer.py`
Model: `meta/llama-4-maverick-17b-128e-instruct` (set via `NVIDIA_MODEL` env var)
Use: default on-demand analysis (~20s)

**Pre-scoring checks injected in `build_system_prompt()`:**

- **Check A — Employment model:** Consulting / staffing / professional services firms → cap `target_alignment` <= 2.0, add blocker.
- **Check B — Domain match:** If required domain is a must-have with no direct CV evidence → cap `relevant_experience` <= 2.0, add blocker. Adjacent experience only counts in `mitigation`.
- **Check C — Explicit must-haves with no CV evidence:** Count hard requirements with zero direct evidence. 2+ unmet → reduce `requirements_coverage` by 1.0 per gap (floor 1.0). 1 unmet → reduce by 0.75. Transferable skills do NOT satisfy hard requirements.

Note: `cache_control: {"type": "ephemeral"}` on system message. Strip this field if switching to Mistral — it rejects it.

### Full pipeline (Ensemble)
File: `ensemble_runner.py`
Scorers (parallel): Maverick + Kimi-K2 + Nemotron-super-49b (thinking=on)
Synthesizer: Nemotron-super-49b (thinking=on)
Use: full analysis (~2min)

`_PRE_SCORING_CHECKS` string is shared by both `SCORER_SYSTEM` and `SYNTHESIS_SYSTEM`.
Synthesizer extra rule: "a model that ignored a pre-scoring check is wrong — override it."

**max_tokens:** scorers = 2000, synthesizer = 2000.

**Kimi-K2 JSON failures:** occasional parse errors — scorer is skipped, synthesis continues with remaining models.

### job_post_parser.py
Providers: Greenhouse, Lever, BambooHR, Ashby, Workday, TeamTailor, SmartRecruiters.
`infer_compensation()` only matches salary-formatted numbers (`[$€£]\d{2,3}[,\s]?\d{3}` or `[$€£]\d{2,3}[kK]`).

### Analysis storage
`STATE_DIR/job-analysis-cache.json` — flat JSON file.
Key: `provider|source_key|job_id`
Value: `{analysis, analyzed_at, run_id}`
`analysis` object must contain `pipeline` field (`"maverick"` or `"ensemble"`).
SQLite migration discussed but not yet done.

---

## LLM model status (as of 2026-05-08)

| Model | Status | Notes |
|---|---|---|
| `meta/llama-4-maverick-17b-128e-instruct` | Active — default scorer | ~6–16s, clean JSON, thin reasoning |
| `moonshotai/kimi-k2-instruct` | Dead — HTTP 410 EOL | |
| `moonshotai/kimi-k2.6` | Active — ensemble scorer | Replacement for kimi-k2-instruct |
| `nvidia/llama-3.3-nemotron-super-49b-v1.5` | Active — ensemble scorer + synthesizer | Requires `nvext: {"thinking": "on"}`. 50–68s. Response in `message.content`, not `reasoning_content` |
| `deepseek-ai/deepseek-v3.2` | Dead — HTTP 410 EOL | |
| `deepseek-ai/deepseek-v4-flash` | Removed — severe timeout on 16k payloads | |
| `mistralai/mistral-large-3-675b-instruct-2512` | Rejects `cache_control` field — strip it to use | |

API base: `https://integrate.api.nvidia.com/v1/chat/completions` (OpenAI-compatible)

---

## Viewer UI

### Active files
- `viewer/public/index.html` — main UI
- `viewer/public/feed.js` — all UI logic
- `viewer/public/feed.css` — all styles
- `viewer/public/saved-searches.json` — search presets (user-editable, not hardcoded in HTML/JS)

### Job card footer buttons
1. **"Quick analyze fit"** (`.btn-analyze--quick`, blue) — Maverick pipeline. After run: label = "Quick — Re-run"
2. **"Full analyze fit"** (`.btn-analyze--full`, purple) — Ensemble pipeline. After run: label = "Full — Re-run"
3. **"JD data"** (ghost, eye icon) — opens JD data modal (fetches `/api/job-parsed`)
4. **"Not interested"** (ghost) — hides card

Score badge (`.btn-score-badge`): 20px pill, appears after analysis, colored blue/purple, clicking opens the side panel.
Both analyze buttons are disabled while either is running. Spinner on active button only.

### Key JS functions
- `footerHtml(job, pipeline)` — full footer HTML, pipeline-aware
- `bindFooterEvents(footer, job)` — all footer event listeners
- `setMainBtnSpinner(job, label, mode)` — disables both buttons, shows spinner
- `restoreMainBtn(job)` — re-renders footer
- `reapplySpinners()` — re-applies spinner state after DOM re-render

### Analysis side panel
- Opens via `.js-open-panel` (score badge)
- Pipeline badge: "Preview · Maverick" (blue, `.pipeline-maverick`) or "Ensemble pipeline" (purple, `.pipeline-ensemble`)
- Panel re-run button uses Maverick mode

### Filter grid
Row 1: job title (full width)
Row 2: location / company / providers / max age
Max age filter uses `first_seen_at` (not `last_seen_at`).

### Queue sidebar
Queue drawer shows items with statuses: `todo`, `running`, `retrying`, `done`, `error`, `permanent_error`.
State managed via `queueDrawerOpen` flag and `activeAnalysisJobs` map.
Queue key format: `provider|source_key|job_id`

### Saved searches
Loaded from `/saved-searches.json` at startup. Rendered dynamically into `#saved-searches-strip`.
Current entries: "TPM, SoFlo" and "PM, SoFlo".
Format: `{id, label, title, location, company, sources, days}`

---

## Hard rules

### Never modify profile files
`matcher/career-ops/_profile.md`, `profile.yml`, `cv.md`, `portals.yml` are user-owned.
All scoring logic goes exclusively in `job_fit_analyzer.py` and `ensemble_runner.py`.

### Discord webhook — URL construction
`companyWebsite()` already returns `https://domain`. Never wrap it again in `https://`.
Guard the embed `url` field: only set it when the value starts with `http://` or `https://`.
Any malformed URL causes Discord to reject the entire payload with 400.

### JD data modal
Uses query params for `/api/job-parsed`, not path params. Required because Workday `source_key` contains `/`.

---

## Benchmark scripts

`matcher/benchmark.py` — single-model benchmark
`matcher/benchmark_ensemble.py` — standalone ensemble benchmark (not the production runner)
`matcher/ensemble_runner.py` — production ensemble runner (JSONL interface)

Candidate profile used: Michael Levy, Senior Technical PM, Cooper City FL.

---

## User stories & backlog

`BACKLOG.md` — prioritized list of 20 planned features (user-visible ordering).
`user-stories/` — one self-contained `.md` per US with full implementation spec.
`user-stories/_context.md` — shared context: file locations, DB patterns, LLM notes, dependency graph between US.

When implementing a US: read `_context.md` first, then the target US file. No need to return to this conversation for context.

---
> Source: [bracketouverte/job-crawler](https://github.com/bracketouverte/job-crawler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
