## demo-lead-scraper

> Turns unstructured search results into structured lead data with AI enrichment.

# AI Lead Scraper + Enricher

Turns unstructured search results into structured lead data with AI enrichment.

**Owner:** Sebastian Rodriguez
**Updated:** 2026-04-14

---

## Product

A small demo app that takes a search query, scrapes DuckDuckGo HTML results, enriches each result with AI, and displays structured lead data in a table.

This is a **sales demo** — part of a multi-demo package showing AI automation capabilities. Not a production scraper. Not a lead gen platform.

**Core loop:** Query -> Scrape DDG -> Parse titles + URLs -> AI enrich -> Render table

**Core transformation:** Unstructured search results → structured lead data

**Business value:** Skip the manual research loop. One query returns structured leads you can act on.

## Stack

| Layer | Tech |
|-------|------|
| App | Next.js (App Router) |
| UI | React + TypeScript + Tailwind |
| API | Next.js API route |
| Scraper | Python + requests + BeautifulSoup4 |
| AI | OpenRouter |

**No** microservices, separate backends, databases, auth, queues, background jobs, crawl depth, pagination, exports, retries infra, fancy design, full lead validation.

## Architecture

- One Next.js app, single page UI + one API route
- Python script called from API route for scraping
- OpenRouter for model enrichment (called from TypeScript)
- No database — everything in-memory per request
- No auth — open demo app

## Codebase

| Path | Purpose | Status |
|------|---------|--------|
| `app/page.tsx` | UI — query input, submit, loading/error, results table | Complete |
| `app/api/leads/route.ts` | API route — calls scraper, enriches, returns leads | Complete |
| `lib/types.ts` | RawLead, Lead, request/response types | Complete |
| `lib/enrich.ts` | OpenRouter enrichment with retry (parallel, paid model) | Complete |
| `lib/mock-leads.ts` | Fallback mock data when scraper fails | Complete |
| `scripts/scrape.py` | Fetch DDG HTML, parse titles + URLs, print JSON | Complete |

## Workspace Layout

```
ai-lead-enricher/              <- YOU ARE HERE (git root)
├── CLAUDE.md                  <- Start here, always
├── Justfile                   <- CLI entry point: just <target>
├── .claude/                   <- Agent definitions + hooks
│   ├── settings.json          <- Hooks (format-on-save, block destructive git)
│   └── agents/                <- robo, architect, backend, frontend, qa
├── .gorp/                     <- Plans, process docs, prompts, journal
│   ├── plans/                 <- current-sprint.md, roadmap.md
│   ├── process/               <- conventions, agent-protocol, approval-matrix
│   ├── prompts/               <- dispatch.md.tmpl
│   └── journal/               <- Per-agent daily logs
├── app/
│   ├── page.tsx               <- Single page UI
│   └── api/leads/route.ts     <- Lead scrape + enrich endpoint
├── lib/
│   ├── types.ts               <- Type definitions
│   ├── enrich.ts              <- OpenRouter enrichment
│   └── mock-leads.ts          <- Fallback mock data
├── scripts/
│   ├── scrape.py              <- DDG scraper
│   ├── dispatch.sh            <- Agent dispatch
│   └── quality-gate.sh        <- Quality checks
├── public/                    <- Static assets
├── .env.example               <- Environment template
├── requirements.txt           <- Python deps
└── README.md                  <- Project overview
```

## Data Model

No database. All data is ephemeral per request.

```
RawLead  { name: string, url: string }
Lead     { name: string, url: string, summary: string, category: string }
```

**RawLead** — output from Python scraper (title + URL from DDG HTML)
**Lead** — enriched by AI model with summary + category

## Views (v1)

1. **Search Page** — Single page with query input, submit button, loading spinner, error display, and results table

That's it. One page.

## UX Rules

- Single page, no navigation
- Query input + submit button at top
- Loading state while scraping + enriching
- Error state if anything fails
- Results table with 4 columns: name, url, summary, category
- 5-10 rows max
- Desktop-first, clean and readable

## Non-Goals

No: auth, database, exports, pagination, background jobs, retries, crawl depth, fancy design, full lead validation, multi-page UI, caching, rate limiting, user accounts

---

## Key Documents

| Doc | Path | Purpose |
|-----|------|---------|
| Launch roadmap | `.gorp/plans/roadmap.md` | Canonical planning reference |
| Conventions | `.gorp/process/conventions.md` | Git, commits, sprint format |
| Agent protocol | `.gorp/process/agent-protocol.md` | Dispatch/report format for agents |
| Approval matrix | `.gorp/process/approval-matrix.md` | What agents can auto-approve vs owner-required |
| Current sprint | `.gorp/plans/current-sprint.md` | Active tasks |
| Gotchas | `.gorp/process/gotchas.md` | Situational issues |

---

## Agent System

This repo uses Claude Code multiagent orchestration. Agents live in `.claude/agents/`.

| Agent | Model | Role | Scope |
|-------|-------|------|-------|
| **robo** | opus | Sprint orchestrator, dispatches others | Plans, coordination |
| **architect** | sonnet | API contracts, type design | Types, contracts |
| **backend** | sonnet | API route, enrichment, scraper integration | `app/api/`, `lib/`, `scripts/` |
| **frontend** | sonnet | Page UI, table, interactions | `app/page.tsx` |
| **qa** | sonnet | Testing, validation | Matches code under review |

### The Build Loop

```
PLAN -> APPROVE -> DISPATCH (waves) -> EXECUTE -> VERIFY -> LOG -> REPLAN
```

1. Owner sets direction in the roadmap
2. `just sprint-plan` — Robo proposes a sprint breakdown
3. `just sprint-approve <name>` — Owner approves
4. `just sprint-run <name>` — Dispatches agents in dependency-ordered waves
5. Agents work, write journal entries
6. QA validates against acceptance criteria
7. `just status` — View sprint results

### Dispatch

```bash
just sprint-plan              # Interactive orchestrator
just sprint-approve phase-1   # Approve a sprint
just sprint-run phase-1       # Full wave-based dispatch
just agent backend            # Single agent interactive
just dispatch backend "task"  # Headless dispatch
just status                   # Check results
```

---

## Prerequisites

- Node.js 20+
- pnpm
- Python 3.10+
- [just](https://github.com/casey/just) — task runner (`brew install just`)

## Local Development

```bash
pnpm install                  # Install JS deps
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt  # Install Python deps
cp .env.example .env.local    # Add LEAD_SCRAPER_OPENROUTER_KEY
pnpm dev                      # Start Next.js dev server
```

---

## Quality Gates

All must pass before merge. Run `just gate` or individually:

| Gate | How |
|------|-----|
| Type check | `pnpm tsc --noEmit` |
| Lint | `pnpm eslint .` |
| Build | `pnpm build` |

---

## Conventions

- **Commits**: `type(scope): description` — scopes: app, api, ui, scraper, enrich, infra, docs
- **Author**: `--author="Gorp, Guava AI <gorp@guava.ai>"`
- **Co-Author**: `Co-Authored-By: Gorp, Guava AI <gorp@guava.ai>`
- **Branches**: `feat/`, `fix/`, `chore/`
- **Sprint tracking**: `.gorp/plans/current-sprint.md`
- **Roadmap**: `.gorp/plans/roadmap.md` (CTO-maintained, agents never modify)
- **Journal**: `.gorp/journal/<agent>-<date>.md`

## Approval Matrix

| Action | Who |
|--------|-----|
| Write code, run tests, create branches | Auto (agents) |
| Task re-prioritization within sprint | Robo |
| Roadmap changes, new deps, scope changes | CTO (Sebastian) |

---

## Rules

- **Agents are proposal engines.** They generate diffs and reports under owner supervision.
- **Every task gets an agent.** No cowboy coding. See conventions.
- **Keep docs short.** If a topic exceeds ~100 lines, split into its own doc and link.
- **Roadmap is canonical.** All planning references `.gorp/plans/roadmap.md`.

## Operating Principles

1. **Propose, don't execute.** Agents generate diffs under owner supervision.
2. **Ask, don't assume.** Query the orchestrator before large-scope changes.
3. **Stay within scope.** Edit only assigned directories.
4. **Respect security boundaries.** Never read/modify `.env` or production credentials.

---

## Launch Status

| Phase | Milestone | Status |
|:-----:|-----------|--------|
| 1 | Project Setup + Scaffold | Complete |
| 2 | Implementation | Complete |
| 3 | Polish + Screenshot | Complete |

## Definition of Done

Done when:
- User enters query
- App returns at least 5 rows
- Each row has all 4 fields (name, url, summary, category)
- Happy path works locally
- Repo has screenshot/gif
- README explains value in under 1 minute
- Positioning aligns with demo package (Demo 1 = lead generation)

## Test Queries

- `miami dental clinics`
- `ecommerce skincare brands`
- `saas payroll startups`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sebastian-O-Rodriguez) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
