## pal

> A personal knowledge agent that learns how you work and builds a compounding knowledge base. Navigates heterogeneous data sources using native query interfaces. Raw data goes in, a structured wiki comes out, and every interaction compounds.

# Pal

A personal knowledge agent that learns how you work and builds a compounding knowledge base. Navigates heterogeneous data sources using native query interfaces. Raw data goes in, a structured wiki comes out, and every interaction compounds.

The canonical specification is `docs/SPEC.md`. All other documentation derives from it.

## Architecture

- Team definition: `pal/team.py` (Pal team leader, Coordinate mode)
- Member agents: `pal/agents/navigator.py` (Navigator), `pal/agents/researcher.py` (Researcher), `pal/agents/compiler.py` (Compiler), `pal/agents/linter.py` (Linter), `pal/agents/syncer.py` (Syncer, conditional)
- Shared settings: `pal/agents/settings.py` (DB, knowledge bases)
- Config: `pal/config.py` (all env var reads and feature flags)
- Instructions: `pal/instructions.py` (prompt strings + builders)
- Tools: `pal/tools/` (knowledge, ingest, wiki, build)
- API server: `app/main.py` (FastAPI + AgentOS + optional Slack interface)
- Custom router: `app/router.py` (/context/reload, /wiki/compile, /wiki/lint, /wiki/ingest, /sync/pull)
- Database: PostgreSQL + pgvector (knowledge, learnings, user data, sessions)

## Team Structure

```
Pal (Team, Coordinate, gpt-5.4)
├── Navigator  — routes queries, reads wiki, handles email/calendar/SQL/files
├── Researcher — web search, source gathering, writes to raw/
├── Compiler   — reads raw/, writes wiki articles, maintains index
├── Linter     — health checks, finds gaps, suggests research
├── Syncer     — commits and pushes context/ changes to GitHub (conditional)
└── [leader responds directly for greetings/simple questions]
```

- **Pal (leader):** Triages requests, delegates to specialists, posts to Slack. Chains Syncer after file-creating workflows.
- **Navigator:** SQLTools, FileTools, MCPTools (Exa), GmailTools, CalendarTools, update_knowledge, wiki read tools, read_manifest
- **Researcher:** FileTools, ParallelTools (parallel_search, parallel_extract), update_knowledge, ingest tools (ingest_url with auto-fetch, ingest_text, read_manifest) (conditional — requires PARALLEL_API_KEY)
- **Compiler:** FileTools, update_knowledge, ingest tools (read_manifest, update_manifest_compiled), wiki tools (read/update index, read/update state)
- **Linter:** FileTools, MCPTools (Exa), update_knowledge, wiki tools (read index, read/update state)
- **Syncer:** sync_push, sync_pull, sync_status (conditional — requires GITHUB_ACCESS_TOKEN + PAL_REPO_URL)

Navigator has both `pal_knowledge` and `pal_learnings`. Researcher, Compiler, and Linter share `pal_knowledge`. Syncer has no knowledge base.

## Key Concepts

- **Navigation over search:** Each source keeps its native query interface. No flattening into a single vector store.
- **Knowledge base pipeline:** Raw data → `context/raw/` → Compiler → `context/wiki/` → Navigator answers questions
- **Dual knowledge system:** `pal_knowledge` (metadata routing) + `pal_learnings` (operational memory). Both PgVector hybrid search.
- **Wiki index as routing layer:** Navigator reads `wiki/index.md` first for knowledge questions, then pulls specific articles.
- **Manifest tracking:** `.manifest.json` in raw/ tracks ingest/compile state. Compiler only processes `compiled: false` entries.
- **Execution loop:** Classify → Recall → Read → Act → Learn
- **Thread-as-session:** Slack thread timestamps = session IDs.
- **Startup schedule registration:** All scheduled tasks register in `app/main.py` lifespan (idempotent, `if_exists="update"`).
- **Git-backed persistence:** Context/ is synced to GitHub via the Syncer agent. No volumes needed — git is the persistence layer. Push is event-driven (after work), pull is scheduled (every 30 min). Requires `GITHUB_ACCESS_TOKEN` + `PAL_REPO_URL`.
- **Production authorization:** `authorization=runtime_env == "prd"` enables RBAC in production. Requires `JWT_VERIFICATION_KEY` from os.agno.com.
- **Governance:** No email sending (draft only). No file deletion. No cross-user data access. External calendar events require confirmation.

## Structure

```
pal/
├── app/
│   ├── main.py              # AgentOS + Slack interface + scheduler + lifespan schedule registration
│   ├── router.py            # /context/reload, /wiki/compile, /wiki/lint, /wiki/ingest, /sync/pull
│   └── config.yaml          # Quick prompts for web UI
├── pal/
│   ├── team.py              # Pal team definition (leader)
│   ├── config.py            # Environment variables and feature flags
│   ├── instructions.py      # Instruction strings + builders
│   ├── paths.py             # Shared path constants
│   ├── agents/
│   │   ├── navigator.py     # Navigator agent (core ops + wiki Q&A)
│   │   ├── researcher.py    # Researcher agent (web → raw/)
│   │   ├── compiler.py      # Compiler agent (raw/ → wiki/)
│   │   ├── linter.py        # Linter agent (wiki health checks)
│   │   ├── syncer.py        # Syncer agent (git commit + push)
│   │   └── settings.py      # Shared DB, knowledge bases
│   └── tools/
│       ├── knowledge.py     # update_knowledge tool
│       ├── ingest.py        # ingest_url, ingest_text, manifest tools
│       ├── wiki.py          # wiki index + state tools
│       ├── git.py           # git sync tools (push, pull, status)
│       └── build.py         # Tool assembly per agent role
├── db/
│   ├── session.py           # PostgreSQL session + knowledge factory
│   ├── url.py               # Database URL builder
│   └── __init__.py          # Public exports
├── context/
│   ├── about-me.md          # User background, goals
│   ├── preferences.md       # Working style, file conventions
│   ├── voice/               # Tone guides (email, slack, x, document)
│   ├── templates/           # Document scaffolds
│   ├── meetings/            # Saved meeting notes
│   ├── projects/            # Project briefs
│   ├── raw/                 # Ingested source material (articles, papers, notes)
│   │   └── .manifest.json   # Ingest/compile state tracking
│   └── wiki/                # LLM-compiled knowledge base
│       ├── index.md         # Master index with article summaries
│       ├── concepts/        # One article per concept
│       ├── summaries/       # One summary per raw document
│       ├── outputs/         # Filed query results and reports
│       └── .state.json      # Compile/lint timestamps and counts
├── evals/
│   ├── run.py               # Unified eval runner (Agno eval framework)
│   ├── test_load_context.py # Unit tests for context loading
│   └── cases/               # Test cases by category
│       ├── security.py      # Never leaks secrets or credentials
│       ├── routing.py       # Leader delegates to right agent/tools
│       ├── governance.py    # Refuses dangerous requests
│       ├── knowledge.py     # Navigates context systems correctly
│       ├── voice.py         # Drafts match voice guides
│       └── wiki.py          # Wiki tools used correctly
├── docs/
│   ├── SPEC.md              # Canonical specification
│   ├── SLACK_CONNECT.md     # Slack app setup guide
│   ├── GOOGLE_AUTH.md       # Google OAuth setup guide
│   └── TEST_CASES.md        # Manual test prompts
├── scripts/
│   ├── entrypoint.sh           # Docker entrypoint
│   ├── google_auth.py          # One-time Google OAuth flow
│   ├── format.sh               # ruff format + import sorting
│   ├── validate.sh             # ruff lint + mypy
│   ├── venv_setup.sh           # Virtual environment setup
│   ├── build_image.sh          # Multi-arch Docker image build
│   ├── generate_requirements.sh # uv pip compile → requirements.txt
│   ├── railway_up.sh           # First-time Railway deployment
│   ├── railway_redeploy.sh     # Lightweight redeploy
│   └── railway_env.sh          # Sync .env to Railway
├── compose.yaml
├── Dockerfile
├── pyproject.toml
└── requirements.txt
```

## Running

```bash
docker compose up -d --build
```

Connect via web UI (os.agno.com → Add OS → Local → http://localhost:8000), Slack (see docs/SLACK_CONNECT.md), or CLI (`python -m pal.team`).

## Setup Flow

1. Clone repo
2. Configure `.env` (`cp example.env .env`, add `OPENAI_API_KEY`)
3. Run locally (`docker compose up -d --build`)
4. Load context metadata (`docker compose exec pal-api python context/load_context.py`)
5. (Optional) Google OAuth for Gmail + Calendar (docs/GOOGLE_AUTH.md)
6. (Optional) Connect to Slack (docs/SLACK_CONNECT.md)

## Local Development

```bash
./scripts/venv_setup.sh && source .venv/bin/activate
docker compose up -d pal-db
python -m pal.team  # CLI mode
```

## Commands

```bash
./scripts/venv_setup.sh && source .venv/bin/activate
./scripts/format.sh                # Format code
./scripts/validate.sh              # Lint + type check
python -m pal.team                # CLI mode (runs test cases)
python context/load_context.py     # Load context metadata
python -m evals                    # Run all evals
python -m evals --category routing # Run single category
python -m evals --verbose          # Show details
python -m evals.test_load_context  # Run unit tests
```

## Scheduled Tasks

All times America/New_York. Results post to `#pal-updates` on Slack if configured.

| Task | Schedule | Cron | Endpoint |
|------|----------|------|----------|
| Context Refresh | Daily 8 AM | `0 8 * * *` | `/context/reload` |
| Daily Briefing | Weekdays 8 AM | `0 8 * * 1-5` | `/teams/pal/runs` |
| Wiki Compile | Daily 9 AM | `0 9 * * *` | `/teams/pal/runs` |
| Inbox Digest | Weekdays 12 PM | `0 12 * * 1-5` | `/teams/pal/runs` |
| Learning Summary | Monday 10 AM | `0 10 * * 1` | `/teams/pal/runs` |
| Weekly Review | Friday 5 PM | `0 17 * * 5` | `/teams/pal/runs` |
| Wiki Lint | Sunday 8 AM | `0 8 * * 0` | `/teams/pal/runs` |
| Sync Pull | Every 30 min | `*/30 * * * *` | `/sync/pull` |

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `OPENAI_API_KEY` | Yes | OpenAI API key (GPT-5.4) |
| `PARALLEL_API_KEY` | No | Parallel web research — enables Researcher agent |
| `EXA_API_KEY` | No | Exa web search for Navigator + Linter (tool loads regardless) |
| `GOOGLE_CLIENT_ID` | No | Gmail + Calendar OAuth (all 3 required together) |
| `GOOGLE_CLIENT_SECRET` | No | Gmail + Calendar OAuth |
| `GOOGLE_PROJECT_ID` | No | Gmail + Calendar OAuth |
| `SLACK_TOKEN` | No | Slack bot token (interface + SlackTools) |
| `SLACK_SIGNING_SECRET` | No | Slack event verification |
| `GITHUB_ACCESS_TOKEN` | No | Git sync — push context/ to GitHub (both required) |
| `PAL_REPO_URL` | No | Git sync — repo URL (both required) |
| `PAL_CONTEXT_DIR` | No | Context directory path (default: `./context`) |
| `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASS`, `DB_DATABASE` | No | PostgreSQL config (defaults: ai/ai@localhost:5432/ai) |
| `PORT` | No | API server port (default: 8000) |
| `RUNTIME_ENV` | No | `dev` for hot reload + local scheduler URL |
| `AGENTOS_URL` | Production | Scheduler callback URL |
| `JWT_VERIFICATION_KEY` | Production | RBAC public key from os.agno.com |

## Database

- **Knowledge:** `pal_knowledge` + `pal_knowledge_contents` (PgVector, hybrid search, text-embedding-3-small)
- **Learnings:** `pal_learnings` + `pal_learnings_contents` (PgVector, hybrid search)
- **Sessions:** `pal_contents` (agent session data)
- **User data:** `pal.*` tables (pal schema — notes, people, projects, decisions, agent-created on demand)
- All user queries scoped to `user_id`. Hard boundary.

---
> Source: [agno-agi/pal](https://github.com/agno-agi/pal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
