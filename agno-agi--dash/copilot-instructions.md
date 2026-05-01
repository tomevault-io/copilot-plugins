## dash

> Dash is a self-learning data agent that delivers **insights, not just SQL results**. It uses a team of specialists (Analyst + Engineer) coordinated by a leader to handle data queries and build computed data assets. Built on [Agno](https://docs.agno.com). Runs in Slack, the terminal, or the [AgentOS](https://os.agno.com) web UI.

# CLAUDE.md

## Project Overview

Dash is a self-learning data agent that delivers **insights, not just SQL results**. It uses a team of specialists (Analyst + Engineer) coordinated by a leader to handle data queries and build computed data assets. Built on [Agno](https://docs.agno.com). Runs in Slack, the terminal, or the [AgentOS](https://os.agno.com) web UI.

## Structure

```
dash/
├── team.py               # Dash team (leader, coordinate mode)
├── settings.py            # Shared config (DB, model, knowledge bases, Slack)
├── instructions.py        # Instruction builders per agent role
├── paths.py               # Path constants
├── __main__.py            # CLI entry point (python -m dash)
├── agents/
│   ├── analyst.py         # SQL queries, data analysis, insights (read-only)
│   └── engineer.py        # Views, summary tables, computed data (dash schema)
├── context/               # Runtime prompt builders (reads knowledge/)
│   ├── semantic_model.py  # Table metadata → system prompt
│   └── business_rules.py  # Business rules → system prompt
└── tools/
    ├── build.py           # Tool assembly per agent role (schema boundaries)
    ├── introspect.py      # Runtime schema inspection (both schemas)
    ├── save_query.py      # Save validated queries to knowledge
    └── update_knowledge.py # Record schema changes to knowledge

knowledge/                 # Data files loaded into vector DB (1:1 mapping)
├── tables/                # Table metadata JSON files (SaaS metrics)
├── queries/               # Validated SQL query patterns
└── business/              # Business rules, metrics, gotchas

app/
├── main.py               # AgentOS entry point (teams, scheduler, Slack interface)
└── config.yaml            # Quick prompts

db/
├── __init__.py            # Re-exports: db_url, get_postgres_db, get_sql_engine, etc.
├── session.py             # PostgreSQL + PgVector + dual schema (public/dash)
└── url.py                 # Database URL builder

evals/                     # Evaluation framework (Agno eval classes)
├── run.py                 # Unified eval runner
└── cases/                 # Test cases by category
    ├── accuracy.py        # AccuracyEval — data correctness
    ├── routing.py         # ReliabilityEval — team routes correctly
    ├── security.py        # AgentAsJudgeEval — no credential leaks
    ├── governance.py      # AgentAsJudgeEval — refuses destructive SQL
    └── boundaries.py      # AgentAsJudgeEval — schema access boundaries

scripts/
├── generate_data.py       # Generate SaaS sample data
├── load_knowledge.py      # Load knowledge into vector DB
├── venv_setup.sh          # Create virtualenv (uses uv)
├── format.sh              # ruff format + import sorting
├── validate.sh            # ruff check + mypy
├── generate_requirements.sh # uv pip compile → requirements.txt
├── build_image.sh         # Multi-platform Docker build
├── entrypoint.sh          # Docker entrypoint (DB wait, banner)
├── railway_up.sh          # First-time Railway setup
├── railway_redeploy.sh    # Redeploy to Railway
└── railway_env.sh         # Sync .env.production to Railway

docs/
├── SLACK_CONNECT.md       # Slack app setup guide with manifest
└── TEST_QUESTIONS.md      # Manual test questions (routing, data quality, edge cases)
```

## Commands

```bash
./scripts/venv_setup.sh && source .venv/bin/activate
./scripts/format.sh      # Format code (ruff format + isort)
./scripts/validate.sh    # Lint + type check (ruff + mypy)
python -m dash           # CLI mode
python -m dash.team      # Test mode (runs sample queries)

# Data & Knowledge
python scripts/generate_data.py      # Generate SaaS sample data
python scripts/load_knowledge.py     # Load knowledge into vector DB

# Evaluations
python -m evals                      # Run all evals
python -m evals --category accuracy  # Run specific category
python -m evals --verbose            # Show response details

# Deployment (uses .env.production)
./scripts/railway_up.sh              # First-time Railway setup
./scripts/railway_redeploy.sh        # Redeploy
./scripts/railway_env.sh             # Sync .env.production to Railway

# Dependencies
./scripts/generate_requirements.sh           # Regenerate requirements.txt
./scripts/generate_requirements.sh upgrade   # Regenerate with latest versions
```

## Architecture

**Dual Schema:**

| Schema | Owner | Access |
|--------|-------|--------|
| `public` | Company (loaded externally) | Read-only — never modified by agents |
| `dash` | Engineer agent | Views, summary tables, computed data |

**Team (Coordinate Mode):**

| Agent | Role | Tools | Schema Access |
|-------|------|-------|---------------|
| **Dash (Leader)** | Routes requests, synthesizes insights | SlackTools (optional) | — |
| **Analyst** | SQL queries, data analysis, insights | SQLTools (read-only, enforced by DB), introspect_schema, save_validated_query, ReasoningTools | Reads public + dash |
| **Engineer** | Views, summary tables, computed data | SQLTools (dash schema), introspect_schema, update_knowledge, ReasoningTools | Reads public, writes dash |

**Interfaces:**

| Interface | Activation | What It Does |
|-----------|------------|--------------|
| **Slack** | `SLACK_TOKEN` + `SLACK_SIGNING_SECRET` | Receives messages via `/slack/events`, streams responses to threads |
| **AgentOS** | Always | Web UI at os.agno.com |

**Knowledge Flow:**

| System | What It Stores | How It Evolves |
|--------|---------------|----------------|
| **Knowledge** | Table metadata, validated queries, business rules, **dash schema objects** | Curated files + Engineer's `update_knowledge` calls |
| **Learnings** | Error patterns, type gotchas, discovered fixes | Managed by LearningMachine (AGENTIC) |

When the Engineer creates a view (e.g., `dash.monthly_mrr`), it calls `update_knowledge` to record the schema, columns, use cases, and example queries. The Analyst discovers these via knowledge search and prefers them over raw table queries.

## Data Model (SaaS Metrics)

Synthetic B2B SaaS dataset (~500 customers, 2 years of data) in `public` schema:

| Table | Description |
|-------|-------------|
| `customers` | Company info, industry, size, acquisition source, status |
| `subscriptions` | Plan, MRR, seats, billing cycle, lifecycle status |
| `plan_changes` | Upgrades, downgrades, cancellations with MRR impact |
| `invoices` | Billing records, payment status, billing periods |
| `usage_metrics` | Daily API calls, active users, storage, reports |
| `support_tickets` | Priority, category, resolution time, satisfaction |

Key gotchas:
- `ended_at` is NULL for active subscriptions
- Annual billing gets 10% discount (amount = mrr * 12 * 0.9)
- Usage metrics are sampled (3-5 days/month), not daily
- `satisfaction_score` is NULL for ~30% of tickets

## Evaluation System

Five eval categories using Agno's eval framework:

| Category | Eval Type | What It Tests |
|----------|-----------|---------------|
| accuracy | AccuracyEval (1-10) | Correct data and meaningful insights |
| routing | ReliabilityEval | Team routes to correct agent/tools |
| security | AgentAsJudgeEval (binary) | No credential or secret leaks |
| governance | AgentAsJudgeEval (binary) | Refuses destructive SQL operations |
| boundaries | AgentAsJudgeEval (binary) | Schema access boundaries respected |

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `OPENAI_API_KEY` | Yes | OpenAI API key |
| `SLACK_TOKEN` | No | Slack bot token (interface + tools) |
| `SLACK_SIGNING_SECRET` | No | Slack signing secret (interface only) |
| `DB_*` | No | Database config (defaults work with Docker Compose) |
| `RUNTIME_ENV` | No | `dev` (hot reload) or `prd` (RBAC enabled) |
| `AGENTOS_URL` | No | Scheduler callback URL (defaults to `http://127.0.0.1:8000`) |
| `JWT_VERIFICATION_KEY` | No | Production RBAC (from os.agno.com) |

## Tooling

- **Python 3.12** (required)
- **uv** — package manager (venv creation, dependency resolution, pip compile)
- **ruff** — linting and formatting (line-length 120)
- **mypy** — type checking (strict-ish: `check_untyped_defs`, `no_implicit_optional`)
- **CI** — GitHub Actions (`.github/workflows/validate.yml`): ruff format check, ruff lint, mypy on every push/PR to main

## Key Patterns

- **Tool factories**: Tools use a closure pattern (`create_*_tool(knowledge)` in `dash/tools/`) — the outer function captures dependencies, the inner `@tool` function is what the agent calls.
- **Instruction composition**: `dash/instructions.py` builds prompts dynamically by concatenating role instructions + semantic model + business context. Leader instructions add/swap Slack sections based on config.
- **Dual knowledge**: `dash_knowledge` (curated: table schemas, queries, rules) vs `dash_learnings` (discovered: error patterns, gotchas). Both are PgVector with hybrid search. The leader only gets learnings; specialists get knowledge.
- **DB re-exports**: `db/__init__.py` re-exports everything from `db.session` and `db.url`. Import from `db` directly (e.g., `from db import get_sql_engine`).
- **Schema enforcement**: Analyst read-only is enforced at the PostgreSQL level (`default_transaction_read_only`). Engineer public-schema writes are blocked by a SQLAlchemy `before_cursor_execute` event listener + regex guard.
- **Eval categories**: Registered in `evals/__init__.py` as a dict mapping name → module path + runner type. Adding a new eval category means adding a case file and a registry entry.

---
> Source: [agno-agi/dash](https://github.com/agno-agi/dash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
