## cfb-database

> College Football Database -- a complete data warehouse for the CFBD (College Football Data) API, powered by Supabase Postgres and dlthub pipelines.

# CFB Database Project

## Project Overview

College Football Database -- a complete data warehouse for the CFBD (College Football Data) API, powered by Supabase Postgres and dlthub pipelines.

**Goals:**
- Ingest all 61 CFBD API endpoints into a well-designed Postgres schema
- Support both analytics (read-heavy, denormalized) and application (normalized, transactional) use cases
- Maintain working pipelines for ongoing 2026 season data
- Full historical data (no storage constraints -- will upgrade Supabase tier if needed)

## Related Projects

This is the schema source of truth for a three-repo college football platform:

| Repo | Role | Relationship |
|------|------|-------------|
| **cfb-app** | Next.js analytics dashboard | Reads from `public`, `core`, `core_staging` schemas |
| **cfb-scout** | FastAPI scouting agent | Reads from `core.roster`, `recruiting.recruits`; owns `scouting` schema |

Schema contract for downstream consumers: `docs/SCHEMA_CONTRACT.md`

**Warning:** Schema changes to `core`, `recruiting`, or `public` schemas can break cfb-app and cfb-scout. Check `SCHEMA_CONTRACT.md` before modifying.

## Tech Stack

| Component | Technology |
|-----------|------------|
| Database | Supabase Postgres |
| Pipelines | dlt (dlthub) with REST API source |
| API Source | CFBD API (collegefootballdata.com) |
| Language | Python 3.11+ |

## Key Constraints

- **75,000 API calls/month (Tier 3)** -- use incremental loading, cache reference data
- **No native pagination** -- CFBD uses year-based filtering, iterate programmatically
- **Storage** -- no hard limit; will upgrade Supabase Pro ($25/mo, 8GB) if needed

## Schema Architecture

The database uses multiple Postgres schemas organized by data domain:

| Schema | Purpose | Examples |
|--------|---------|---------|
| `ref` | Reference/lookup data | teams, venues, conferences |
| `core` | Normalized game data | roster, games, game_team_stats, line scores |
| `stats` | Player/team statistics | player_stats, team_stats |
| `ratings` | Rankings and ratings | SP+, Elo, FPI, SRS |
| `recruiting` | Recruiting data | recruits, team_recruiting |
| `betting` | Betting lines | lines, spreads |
| `draft` | NFL draft data | picks, positions |
| `metrics` | Advanced metrics | PPA, win probability |
| `analytics` | Computed analytics | EPA, style profiles |
| `marts` | Materialized views (19) | Denormalized, query-optimized |
| `api` | API view layer (7) | Contract surface for cfb-app/cfb-scout |
| `public` | Convenience views/RPCs (8) | Downstream consumer interface |

### Marts System

19 materialized views in the `marts` schema provide denormalized, query-optimized data. Refresh via:
```bash
python scripts/refresh_marts.py        # Refresh all marts
```
Or use the `refresh_all_marts()` RPC.

### dlt Table Conventions

Tables loaded via dlt use parent-child relationships with `_dlt_id`, `_dlt_parent_id`, `_dlt_list_idx` columns for nested data traversal.

## Project Structure

```
cfb-database/
├── CLAUDE.md
├── pyproject.toml                # Dependencies, CLI entry point, ruff/pytest config
├── docs/
│   ├── SCHEMA_CONTRACT.md        # API surface contract for downstream repos
│   ├── pipeline-manifest.md      # Endpoint-to-pipeline status (source of truth)
│   ├── cfbd-api-endpoints.md     # Complete CFBD API reference
│   ├── dlt-reference.md          # Pipeline configuration patterns
│   ├── SPRINT_PLAN.md            # Implementation plan
│   ├── plans/                    # Dated sprint/implementation plans (22 files)
│   ├── brainstorms/              # Design exploration docs
│   └── solutions/                # Solved problem documentation
├── src/
│   ├── pipelines/
│   │   ├── config/               # RESTAPIConfig definitions
│   │   │   ├── endpoints.py
│   │   │   └── years.py
│   │   ├── sources/              # 14 endpoint-specific source modules
│   │   ├── utils/
│   │   │   ├── api_client.py     # Custom CFBD HTTP client (httpx)
│   │   │   └── rate_limiter.py   # Monthly budget tracking
│   │   └── run.py                # Pipeline orchestration
│   └── schemas/
│       ├── api/                  # 7 API view definitions (contract surface)
│       ├── functions/            # SQL functions
│       ├── marts/                # 19 materialized view definitions
│       ├── public/               # 8 convenience views + RPCs
│       └── migrations/           # Schema migrations
├── scripts/
│   ├── refresh_marts.py          # Mart refresh script
│   └── run_migrations.py         # Migration runner
├── tests/                        # 8 test files + test_sources/
│   └── conftest.py               # DB connection + mock fixtures
├── .dlt/
│   ├── config.toml               # Runtime config (workers, year ranges, rate limits)
│   └── secrets.toml              # API keys (not committed)
└── .githooks/pre-push            # Runs ruff + pytest before push
```

## Key Documentation

| File | Purpose |
|------|---------|
| `docs/SCHEMA_CONTRACT.md` | Defines the public API surface for cfb-app and cfb-scout |
| `docs/pipeline-manifest.md` | Single source of truth for endpoint-to-pipeline mappings with row counts and statuses |
| `docs/cfbd-api-endpoints.md` | Complete CFBD API reference |
| `docs/dlt-reference.md` | Pipeline configuration patterns |

## Skills & Guidelines

### Postgres Best Practices (Supabase)
Location: `~/.claude/skills/postgres-best-practices/`

Apply these guidelines when designing schema and writing queries:

**CRITICAL priority:**
- Add indexes on WHERE and JOIN columns
- Use connection pooling (Supabase provides this)
- Enable RLS for any user-facing tables

**HIGH priority:**
- Use `bigint generated always as identity` for primary keys
- Index all foreign key columns
- Choose appropriate data types (`timestamptz`, `text`, `numeric`)
- Use lowercase snake_case identifiers

**MEDIUM priority:**
- Batch INSERT statements for bulk loading
- Use cursor-based pagination, not OFFSET
- Use UPSERT (`ON CONFLICT`) for idempotent loads

**For large tables (plays, drives, player_stats):**
- Partition by season for plays table (millions of rows)
- Use BRIN indexes for time-series data (created_at, game_date)
- Use partial indexes for filtered queries (e.g., active season only)

### CFBD API Skill
Location: `~/.claude/skills/cfbd-api/`

Use this skill for CFBD API reference when building or debugging pipelines:

- **All 15 API classes** with method signatures and common parameters
- **Authentication pattern**: Bearer token via `cfbd.Configuration(access_token=...)`
- **Endpoint coverage**: Games, teams, plays, drives, stats, ratings, rankings, recruiting, betting, metrics, players, coaches, conferences, venues, draft
- **Common parameters**: `year` (required for most), `week`, `team` (full name, e.g. "Ohio State"), `conference` (abbreviation, e.g. "SEC", "B1G")
- **Gotchas**: Cloudflare burst blocking (~10 min), nullable fields, v2 breaking changes from v1

When adding new pipeline sources or debugging API responses, reference this skill for correct method names, parameter types, and response models.

### dlt REST API Source Skill
Location: `~/.claude/skills/dlt-rest-api/`

Use this skill when building or modifying dlt pipelines:

- **RESTAPIConfig structure**: `client`, `resource_defaults`, `resources` — full field reference
- **Authentication**: Bearer, API Key, HTTP Basic, OAuth2 — both dict syntax (`"type": "bearer"`) and class imports (`BearerTokenAuth`)
- **Pagination**: All 7 types (json_link, header_link, offset, page_number, cursor, header_cursor, single_page) with constructor params
- **Incremental loading**: Placeholder syntax (`{incremental.start_value}`), cursor_path, initial_value
- **Parent-child resources**: Path placeholders (`{resources.parent.field}`), query params, legacy resolve syntax
- **Write dispositions**: `append`, `replace`, `merge` — when to use each
- **Config/secrets**: `.dlt/config.toml`, `.dlt/secrets.toml`, env var patterns (`SOURCES__*`)
- **Gotchas**: auto-pagination failures, data_selector for wrapped responses, nested table unnesting, state per pipeline_name

Reference this skill when writing `RESTAPIConfig` dicts, debugging pagination, configuring incremental loads, or setting up new source modules.

## CFBD API Categories

| Category | Endpoints | Refresh Strategy |
|----------|-----------|------------------|
| Reference (teams, venues, conferences) | 10 | Full refresh (rarely changes) |
| Games & Schedules | 9 | Incremental by year |
| Plays & Drives | 6 | Incremental by year (largest data) |
| Stats (player/team) | 6 | Incremental by year |
| Ratings (SP+, Elo, FPI) | 5 | Incremental by year |
| Recruiting | 3 | Incremental by year |
| Betting Lines | 1 | Incremental by year |
| Metrics (PPA, win prob) | 8 | Incremental by year |
| Draft | 3 | Incremental by year |

## Commands

```bash
# Setup
python -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"

# Run pipelines
python -m src.pipelines.run --source reference           # Load reference data
python -m src.pipelines.run --source games --year 2024   # Load games for a year

# Marts
python scripts/refresh_marts.py     # Refresh all materialized views

# Migrations
python scripts/run_migrations.py    # Apply pending migrations

# Testing & linting
.venv/bin/ruff check .              # Lint
.venv/bin/ruff format --check .     # Format check
.venv/bin/pytest -q                 # Run tests
```

## Testing

- `tests/` contains 8 test files and a `test_sources/` subdirectory
- `conftest.py` provides DB connection fixtures and mock fixtures
- Integration tests hit live Supabase (`test_api_views.py`, `test_marts.py`, `test_player_analytics.py`)
- Unit tests for config, rate limiter, year ranges
- Pre-push hook (`.githooks/pre-push`) runs `ruff check`, `ruff format --check`, `pytest`

## Environment Variables

Credentials live in `.dlt/secrets.toml` (not `.env`). See `.dlt/secrets.toml.example`.

```bash
# .dlt/secrets.toml
[sources.cfbd]
api_key = ""                # From collegefootballdata.com/key

[destination.postgres]
credentials = ""            # Supabase direct connection string

# .env (non-secret config only)
SUPABASE_DB_URL=            # Direct Postgres connection string
```

## Git Conventions

- Branch names: `feature/`, `fix/`, `refactor/`, `chore/` prefixes
- Commit messages: imperative mood, 50-char subject line

## Data Availability Notes

Historical depth varies by endpoint:
- Games/scores: 1869+
- Play-by-play: ~2004+
- Recruiting: ~2000+
- Advanced metrics (SP+, PPA): ~2014+

**Load strategy:** Full history where available. Play-by-play (2004+) will be the largest table -- estimate 5-10M rows, partitioned by season.

## Configuration

- `pyproject.toml` -- project metadata, dependencies, CLI entry point (`cfb-pipeline`), ruff config (line-length 100, py311), pytest config
- `.dlt/config.toml` -- runtime config: worker count, file chunking, year ranges per source, monthly API budget (75,000)
- Supabase MCP server enabled in `.claude/settings.local.json` for interactive SQL during development

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rstover-fo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rstover-fo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
