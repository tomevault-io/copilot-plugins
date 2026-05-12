## gpilot

> > **New here?** Read [README.md](README.md) first, then [docs/getting-started.md](docs/getting-started.md).

# GPilot — Financial Intelligence System

> **New here?** Read [README.md](README.md) first, then [docs/getting-started.md](docs/getting-started.md).
> **To customize**: Copy `.env.example` to `.env`, fill in your details, then run `bash scripts/customize.sh`.

## Identity

- **Operator**: YN
- **Organization**: Acme Ventures
- **Focus Sectors**: Software/SaaS, AI/ML, Fintech, Healthtech, Deeptech
- **Primary output**: Bilingual research reports (EN→CN), investment memos, portfolio analysis

## Directory Map

```
.
├── CLAUDE.md           ← You are here (system brain)
├── raw/                ← Source ingestion inbox (human drops files, LLM never modifies)
│   ├── deals/          ← Term sheets, deal flow, broker quotes
│   ├── research/       ← Articles, papers, conference notes
│   ├── market-intel/   ← News clips, analyst notes
│   ├── meetings/       ← Transcripts, notes
│   └── images/         ← Charts, diagrams for research
├── wiki/               ← LLM-compiled knowledge graph
│   ├── _index.md       ← Master TOC
│   ├── _summaries.md   ← 2-3 line summary of every document
│   ├── companies/      ← Per-company articles
│   ├── sectors/        ← Sector analysis articles
│   ├── deals/          ← Deal pipeline + per-deal status
│   ├── people/         ← Counterparties, network profiles
│   └── concepts/       ← Frameworks, structures, regulations
├── data/
│   ├── fund/           ← Workbooks (deal-tracker, portfolio, etc.)
│   ├── comps/          ← Sector comparable models
│   └── state/          ← JSON state (canonical SSOT) + schema
├── deals/              ← Per-company folders (notes, models, memos)
│   ├── {company-a}/    ├── {company-b}/    └── {company-c}/
├── learnings/          ← Agent processing experience (self-evolving)
│   ├── _index.md       ← Learnings overview
│   ├── preferences.md  ← User preferences (auto-learned)
│   ├── system.md       ← Cross-cutting system learnings
│   └── {agent}.md      ← Per-agent learnings (9 files)
├── agents/             ← 9 AI agent definitions
├── commands/           ← 22 slash command workflows
├── skills/             ← 7 skill domains (with reference docs)
├── templates/          ← 8 document templates
├── scheduled/          ← 10 core scheduled tasks
├── modules/            ← Optional modules (fund-ops, extras, integrations)
├── output/             ← Generated deliverables
├── research/           ← Research pipeline (wip/ + published/)
├── config/             ← Global config, rules, MCP settings
├── dashboard/          ← Next.js portfolio dashboard
├── scripts/            ← Setup and customization scripts
└── docs/               ← Getting started guide
```

## Data Sources

| Source | Access | Content |
|--------|--------|---------|
| **Local `data/`** | Direct read | Workbooks, sector comp models, state JSON |
| **Local `deals/`** | Direct read | Per-deal analysis, models, memos |
| **Google Drive** | MCP or symlink via `projects/` | Cloud-stored documents |
| **OneDrive** | MCP or symlink via `projects/` | Cloud-stored documents |

## Agents

| Agent | Role |
|-------|------|
| `deep-researcher` | Multi-source research via Perplexity MCP |
| `market-researcher` | TAM/SAM, competitive landscape, market maps |
| `financial-analyst` | Modeling, comps, valuations + public market data |
| `deal-sourcer` | Proactive opportunity discovery from GitHub, funding rounds, reports |
| `portfolio-monitor` | Portfolio news, KPI tracking, early warnings |
| `memo-writer` | Investment memos and financial documents |
| `editor` | Quality review, fact-checking, style enforcement |
| `translator` | EN↔CN bilingual (cultural adaptation) |
| `data-visualizer` | Charts, tables, frameworks for research |

> Additional agent in `modules/fund-ops/`: `legal-reviewer` (term sheet + side letter analysis)

## Commands

### Research Pipeline
| Command | Purpose |
|---------|---------|
| `/research` | Full research publication pipeline (8-step, with outline review) |
| `/research-fast-track` | Streamlined 6-step for pre-approved weekly/monthly publications |
| `/publish` | Format for WeChat + LinkedIn distribution |
| `/query` | Research a question against the wiki knowledge base |
| `/company-intel` | Deep company intelligence brief |

### Deal Sourcing & Analysis
| Command | Purpose |
|---------|---------|
| `/source-deals` | Proactive opportunity discovery from public sources |
| `/deal-screen` | Evaluate and score an investment opportunity |
| `/ic-memo` | Generate investment memo |
| `/portfolio-review` | Investment portfolio health check |
| `/board-prep` | Board/advisory meeting materials |
| `/market-data` | Public market data lookup |
| `/earnings-watch` | Earnings analysis |
| `/dcf` | Build institutional-quality DCF model with Bear/Base/Bull + sensitivity tables |
| `/portfolio-variance` | Per-company variance analysis — actuals vs budget, covenant compliance, KPI trends |
| `/founder-outreach` | Draft personalized founder cold email (Gmail drafts only, mandatory email dedup) |

### Knowledge Base & System
| Command | Purpose |
|---------|---------|
| `/ingest` | Process raw/ files into wiki (use `--full` for complete rebuild) |
| `/lint-wiki` | Wiki health check: stale data, broken links, contradictions |
| `/weekly-digest` | Weekly intelligence summary |
| `/sync` | Data synchronization across state files |
| `/reflect` | Agent self-reflection — review recent work, capture learnings |
| `/review-learnings` | Curate accumulated learnings — prune, validate, promote |
| `/jobs` | List, resume, or clean up multi-session running jobs |

> Additional commands in `modules/fund-ops/`: `/lp-quarterly`, `/capital-call`, `/compliance-check`

## Research Publication Calendar

| Type | Cadence | Day | Format |
|------|---------|-----|--------|
| Market Note 市场脉搏 | Weekly | Wed | Bilingual |
| Company Teardown 独角兽拆解 | Bi-weekly | Mon | Bilingual |
| Insight Letter 洞见 | Monthly (wk1) | Thu | Bilingual |
| Thematic Research 主题研究 | Monthly (wk3) | Mon | Bilingual |
| Sector Deep Dive 赛道纵深 | Quarterly | Mid-Q | Bilingual |

Distribution: WeChat (CN) + LinkedIn (EN)

### Research Pipeline Tiers

| Tier | Publication Types | Approval Gates | Steps |
|------|-------------------|----------------|-------|
| **Fast-track** | Market Note, Insight Letter | Pre-approved topic list + final review only | 6 steps via `/research-fast-track` |
| **Full process** | Sector Deep Dive, Company Teardown, Thematic | Outline review + final review | 10 steps via `/research` |

Fast-track requires a monthly pre-approved topic list in research-tracker. Falls back to full process if editorial score < 4.0/5.

## Scheduled Tasks (10 core)

**Daily**: morning-briefing, daily-email-summary, news-briefing, deal-flow-triage, portfolio-news
**Weekly**: deal-sourcing (Wed), weekly-pipeline (Mon), weekly-market-note (Mon), research-calendar-check (Mon), editorial-calendar

> Additional tasks in `modules/fund-ops/scheduled/` (5) and `modules/extras/scheduled/` (11)

## Safety Rules

1. **Email**: ALL Gmail actions produce DRAFTS ONLY. Never auto-send.
2. **Financial Data**: All xlsx modifications must be verifiable.
3. **External Communications**: Always require user review before any external send.
4. **Valuations**: Flag methodology and confidence level on all changes.
5. **Compliance**: Escalate overdue items with increasing urgency.
6. **Confidentiality**: Never expose portfolio company details or deal terms publicly.
7. **Publications**: All research requires user review before publishing.
8. **Portfolio References**: Anonymize in public research unless explicitly approved.

## State Layer (data/state/)

JSON is the canonical machine-readable state. xlsx in `data/fund/` remains as human-readable views.

| File | Purpose | xlsx Counterpart |
|------|---------|-----------------|
| `deals.json` | Deal pipeline | deal-tracker.xlsx |
| `portfolio.json` | Portfolio companies | portfolio-dashboard.xlsx |
| `research.json` | Research pipeline | research-tracker.xlsx |
| `public_holdings.json` | Public positions | public-companies.xlsx |
| `watchlist.json` | Watch list | watchlist.xlsx |
| `running-jobs.json` | Multi-session job tracker | — |
| `evolution-log.json` | Agent learning audit trail | — |
| `schema.json` | JSON Schema for all above | — |

**Read/write JSON first**, then optionally regenerate xlsx via `sync-state-to-xlsx.md`. JSON is git-diffable and avoids binary merge conflicts.

## Knowledge Base (Karpathy Pattern)

The wiki is a **living, LLM-compiled knowledge graph**. The human drops sources into `raw/`, the LLM compiles them into `wiki/`, and queries feed back into the wiki over time.

### Ownership
- `raw/` — **human's domain**. Source documents. LLM never modifies.
- `wiki/` — **LLM's domain**. Human rarely edits directly. LLM maintains, links, lints.
- `output/queries/` — query answers. Valuable ones get filed back into wiki.

### Workflow
1. Human drops file into `raw/deals/`, `raw/research/`, etc.
2. Run `/ingest` — LLM reads raw, updates wiki articles, maintains `_index.md` and `_summaries.md`
3. Run `/query "question"` — LLM researches against wiki, supplements with web, saves answer
4. Run `/lint-wiki` weekly — find stale data, broken links, missing entities
5. Run `/ingest --full` occasionally — full rebuild when wiki quality degrades

### The Compounding Effect
Every query adds to the wiki. Week 1: 5 company articles. Month 3: cross-cutting insights emerge automatically. The wiki becomes the competitive advantage — it knows deal flow history, market patterns, counterparty track records.

### Learnings vs. Wiki

The wiki stores **objective domain knowledge** ("Company X raised $50M Series C"). Learnings store **processing experience** ("When searching for Series C rounds, include the term 'growth round' to catch non-standard naming"). The wiki helps agents know *what*; learnings help agents know *how*. Both compound over time, but learnings are agent-specific while wiki is shared.

## Agent Evolution Framework

GPilot agents are self-evolving. They learn from mistakes, accumulate processing experience, and improve over time — zero external dependencies, all file-based.

### Three-Tier Memory

| Tier | Location | What | Who Writes | When Read |
|------|----------|------|-----------|-----------|
| **Preferences** | `learnings/preferences.md` | User corrections, output preferences | Auto after user corrections (2+) | Session start |
| **Wiki** | `wiki/` | Objective domain facts | `/ingest`, `/query` feedback loop | During research |
| **Learnings** | `learnings/{agent}.md` | Processing experience, tool tips, workflow tricks | Agent reflection after tasks | Agent startup |

### Session Startup Protocol

At the beginning of every session:
1. Check `data/state/running-jobs.json` — are there jobs in progress from previous sessions?
2. If yes, inform the user: "I see {N} jobs in progress: {list}. Want to continue one?"
3. Read `learnings/preferences.md` for user preferences
4. When launching any agent, that agent reads its own `learnings/{agent}.md` first

### Reflection Protocol

After completing any multi-step task (commands, agent workflows):
1. Agent self-assesses: what went well, what was unexpected, what was a workaround?
2. If a reusable insight was gained, append to `learnings/{agent}.md`
3. If user corrected output, note the correction pattern in `learnings/preferences.md`
4. Update `data/state/running-jobs.json`: mark job completed or update progress
5. If the task produced wiki-worthy facts, suggest wiki updates (existing pattern)

### Running Jobs

Multi-session jobs are tracked in `data/state/running-jobs.json`. Commands like `/research`, `/source-deals`, and `/deal-screen` that may span sessions should:
- Register a job entry when they start (status: in_progress)
- Update progress as steps complete
- Mark completed when finished
- Mark abandoned if the user cancels

### Evolution Tracking

Agent improvement events are logged to `data/state/evolution-log.json`. This enables periodic review via `/review-learnings` of what agents have learned and how effective their learnings are.

### Evolution Loop

```
Agent startup → reads learnings (past experience)
             → reads wiki (domain knowledge)
             → reads state (current situation)
             → executes task
             → auto-updates wiki/state
             → agent reflects → appends learnings
             → next session auto-loads this experience
```

## Working Conventions

- **Knowledge Base**: Drop sources into `raw/`, run `/ingest`. Query with `/query`. Lint with `/lint-wiki`.
- **Wiki**: Read `wiki/_index.md` first for any research task. The wiki is the primary knowledge source.
- **State**: Read/write `data/state/*.json` for machine operations. Use `data/fund/*.xlsx` for human review.
- **Deals**: Check `deals/{company}/` for local analysis AND `projects/` for cloud materials.
- **Comps**: Use `data/comps/` sector models for valuations and market analysis.
- **xlsx**: Read `data/fund/` workbooks via the `xlsx` skill.
- **Documents**: Use `templates/` for generation. Use `docx` skill for memos, `pdf` for deliverables.
- **Research**: English first → translate via `skills/translator/` skill → review via `skills/editor/` skill.
- **Deep research**: Use `skills/deep-researcher/` skill (Perplexity MCP primary) over basic WebSearch.
- **Fast-track**: For pre-approved Market Notes and Insight Letters, use `/research-fast-track` instead of `/research`.
- **Learnings**: Agents read `learnings/{agent}.md` on startup. Append new learnings after each task via the Reflection Protocol.
- **Running Jobs**: Check `data/state/running-jobs.json` at session start for in-progress work.
- **Preferences**: Check `learnings/preferences.md` for user output and workflow preferences.
- **Cross-platform**: If symlinks fail (Linux/Windows), use MCP tools per `projects/ACCESS.md`.

## Cross-Machine Sync

| Machine | Role | Sync |
|---------|------|------|
| Primary desktop | Automation hub, cron jobs | Git pull (scheduled) |
| Laptop | Daily work | Git push/pull (manual) |
| Secondary | Backup | Git pull |

---
> Source: [ruiyang-xu/GPilot](https://github.com/ruiyang-xu/GPilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
