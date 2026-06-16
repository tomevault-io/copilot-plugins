## job-search-automation

> This repo is a personal job-search pipeline. It scrapes job boards, scores listings with an LLM, and serves a local dashboard for manual review and pipeline tracking. Application submission and applied status are human-controlled.

# Claude Instructions

This repo is a personal job-search pipeline. It scrapes job boards, scores listings with an LLM, and serves a local dashboard for manual review and pipeline tracking. Application submission and applied status are human-controlled.

## Context system

`.context/` is the persistent context layer for Claude Code. It tells Claude what exists and where to find it. The source of truth for career data lives in `data/`.

The repo ships example copies at `.context.example/` and `data.example/`. To use them, run:

```bash
cp -r .context.example .context
./scripts/setup.sh
```

Then edit both to reflect you. `.context/` and `data/` are gitignored.

**At session start, always read:**
- `.context/people/applicant.md` — who the applicant is, working preferences
- `.context/people/voice.md` — writing rules (critical for anything in the applicant's voice)

**When working in the pipeline code, also read:**
- `.context/projects/job-search.md` — architecture, features, pipeline setup
- `.context/reference/dashboard-files.md` — file map for what to edit
- `.context/decisions/architecture.md` — why things are built this way

**When doing interview prep, outreach, or application work, also read:**
- `data/context.md` — full career context, preferences, deal breakers
- `data/career-detail.md` — deep project documentation with honest assessments
- `data/experience/*.md` — per-company experience files
- `.context/reference/interviews.md` — interview patterns and learnings

**When answering application questions or writing cover letters: write the answer first, analysis after.** Don't evaluate the role or editorialize before drafting the answer. If there are concerns about fit, put them after.

## Writing rules (applies to ALL output)

**Never use em dashes, en dashes, or hyphens as sentence connectors.** Rewrite with a comma or period instead. No exceptions, anywhere in the output.

## Slash commands

The following commands are available in both Claude Code (`.claude/commands/`) and Codex (`.codex/skills/`):

| Command | When to use |
|---|---|
| `/load-context` | Start of any job-search or career session. Reads `.context/` and the active profile. |
| `/job-search` | Review pending listings, approve/reject, check stats, run the pipeline. |
| `/interview-prep [company, email, or JD]` | Full interview prep: tell me about yourself, behavioral stories, match points, tech areas to review, questions to ask. Works for phone screens and second rounds. |
| `/app-questions [question text]` | Answer application form questions in the applicant's voice. |
| `/save-context` | End of session. Persist anything learned back to `.context/`. |

Claude Code also has two import helpers (these live only in `.claude/commands/`, not Codex):

| Command | When to use |
|---|---|
| `/add-job [paste LinkedIn JD]` | Parse a pasted LinkedIn listing and create a scored record in the jobs DB. |
| `/add-company [name or careers URL]` | Add a specific employer to the pipeline. Finds and verifies its ATS board (Greenhouse/Ashby/Lever/Workday) via the go-backend `add-company` subcommand, then it is scraped on the next run. Good for large or legacy employers that automated discovery misses. |

All commands read profile files from `data/`. In Docker, SQLite lives in the named volume mounted at `/app/db`; outside Docker it falls back to `data/jobs.db`.

## Git workflow

Prefer small, reviewable commits with clear messages. Stage only files relevant to the task.

Commit message format: `<type>: <brief summary>` — types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`.

**Never commit:**
- `jobs.json` (auto-generated)
- `jobs.db` (per-profile SQLite, local fallback only)
- `.env` files
- Auto-generated build artifacts (`*.pdf`)
- `market-research-cache.json`, `slug-health.json`, or any `*-cache.json`
- Personal content under `.context/` or `data/` (both gitignored)

**Auto-push and PR:** A Stop hook (`.claude/hooks/auto-pr.sh`) runs at the end of every session. If the feature branch has commits ahead of `dev`, it pushes, opens a PR **into `dev`** (not `main`), and **auto-merges** it into `dev`. `main` is never touched automatically; promoting `dev` to `main` stays a deliberate, manual step. No manual `/ship` needed.

---
> Source: [jakemercure28/job-search-automation](https://github.com/jakemercure28/job-search-automation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
