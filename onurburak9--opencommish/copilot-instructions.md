## opencommish

> Yahoo NBA Fantasy Basketball analytics platform that collects daily stats, enriches them with computed insights (missed lineup opportunities, projection accuracy, achievements), generates LLM-driven Turkish-language recaps, and will serve interactive dashboards through a web application.

# AGENTS

## Project overview

Yahoo NBA Fantasy Basketball analytics platform that collects daily stats, enriches them with computed insights (missed lineup opportunities, projection accuracy, achievements), generates LLM-driven Turkish-language recaps, and will serve interactive dashboards through a web application.

**League:** "teletabi ligi" (ID: 93905, 8 teams, 2025 NBA season)

**Architecture:** See [ARCHITECTURE.md](ARCHITECTURE.md) for the full codemap, invariants, and system design.

**Data Pipeline** (production, runs daily via GitHub Actions):
- `cron/fetch_daily_stats.py` — Actual player stats from Yahoo API (3AM PST)
- `cron/fetch_projected_stats.py` — Projected stats scraped from Yahoo (12PM PST)
- `cron/enrich_daily_data.py` — Merges actuals + projections, computes missed opportunities, achievements, awards
- `cron/fetch_league_context.py` — League standings, matchup scores, week metadata
- `cron/generate_recap.py` — Orchestrator: enriches data, calls Claude API, outputs Turkish recap

**Web Application** (planned):
- `backend/` — FastAPI + PostgreSQL REST API
- `frontend/` — Next.js + TypeScript + Tailwind CSS dashboard

**Supporting:**
- `tests/unit/` — pytest unit tests for enrichment logic
- `tests/integration/` — Playwright browser tests
- `llm/daily-recap-prompt.md` — System prompt for Turkish-language daily recaps
- `dashboard/` — Streamlit dashboard (being replaced by `frontend/`)

## Global rules

- Always run existing tests before committing: `python -m pytest tests/unit/ -v -o "addopts="`
- Follow existing patterns — copy from similar code rather than inventing new approaches
- Update `ARCHITECTURE.md` when making structural changes (adding services, changing data flows, new directories)
- When modifying cron scripts, verify they still work as standalone CLIs: `python cron/<script>.py YYYY-MM-DD`
- After every `git push` to a PR branch, check CI and iterate until green
- Use `uv` for Python package management when available, fall back to `pip`
- If commands are not found, run `source ~/.zshrc` to ensure PATH is loaded
- Never commit `.env` or files containing API credentials
- Data files in `data/` are committed to main by GitHub Actions — never manually edit them

## Coding standards

### Python (Backend + Cron)

- PEP 8 style, type hints on all function signatures
- Use `async/await` for API endpoints and database operations
- Group imports: standard library, third-party, local
- Use FastAPI's `HTTPException` for API errors
- Cron scripts must be standalone CLIs with `YYYY-MM-DD` argument

### TypeScript (Frontend)

- Strict mode enabled
- Components: PascalCase (`TeamDashboard.tsx`)
- Functions/variables: camelCase (`fetchTeamData`)
- Types/Interfaces: PascalCase (`TeamStats`, `MatchupData`)
- Functional components with hooks, no class components

### Database

- Alembic for all schema changes
- Tables: plural snake_case (`player_stats`, `team_rosters`)
- Columns: snake_case (`created_at`, `team_id`)
- Indexes: `ix_{table}_{column}`
- Foreign Keys: `fk_{table}_{referenced_table}`

### API Design

- All endpoints under `/api/v1/`
- RESTful HTTP methods (GET, POST, PUT, DELETE)
- Consistent JSON response structure
- Pagination with limit/offset for list endpoints

## Yahoo Fantasy API notes

- **Library:** yfpy (v17.0+) with `game_code='nba'`
- **Auth:** OAuth 2.0, tokens in env vars, auto-refresh via yfpy
- **Key methods:** `get_team_roster_player_info_by_date()`, `get_league_scoreboard_by_week()`, `get_league_standings()`, `get_game_weeks_by_game_id()`
- **Rate limiting:** Yahoo has undocumented rate limits — scripts include retry logic
- **Stat modifiers:** PTS=1.0, REB=1.2, AST=1.5, ST=3.0, BLK=3.0, TO=-1.0

## Roster position logic (critical domain concept)

**Active positions** (count toward team score): PG, SG, G, SF, PF, F, C, Util

**Inactive positions** (do NOT count): BN (Bench), IL (Injured List), IL+ (Injured List Plus)

**Missed opportunity** = inactive player scored more than a swappable active player at a compatible position. Position compatibility is defined in `enrich_daily_data.POSITION_COMPATIBILITY`. IL/IL+ swaps require a roster transaction and are flagged `swap_feasible: false`.

A player with no game that day (empty `opponent` field) is never a missed opportunity, regardless of position.

## Environment variables

```bash
# Yahoo API (required for data collection)
YAHOO_CLIENT_ID=...
YAHOO_CLIENT_SECRET=...
YAHOO_ACCESS_TOKEN=...
YAHOO_REFRESH_TOKEN=...
YAHOO_TOKEN_TIME=...
YAHOO_TOKEN_TYPE=bearer
YAHOO_GUID=None

# BallDontLie API (optional, for NBA opponent lookup)
BALLDONTLIE_API_KEY=...

# LLM (required for recap generation)
ANTHROPIC_API_KEY=...

# Database (required for backend)
DATABASE_URL=postgresql://postgres:password@localhost:5432/opencommish

# Backend/Frontend URLs
BACKEND_URL=http://localhost:8000
FRONTEND_URL=http://localhost:3000
```

## Implementation phases

### Phase 0: Data Pipeline (COMPLETE)

What exists today and runs in production:

- Daily stats collection from Yahoo API via GitHub Actions
- Projected stats collection (scraping + API)
- Data enrichment: missed opportunities, achievements, projection diffs, daily awards
- LLM-powered Turkish recap generation via Claude API
- Streamlit dashboard (functional, being replaced)
- 60+ days of collected data (Feb-Apr 2026)
- Unit tests for enrichment logic (21 tests passing)

### Phase 1: Backend Foundation

Build the FastAPI backend and migrate from file-based storage to PostgreSQL.

- FastAPI project scaffold with proper project structure
- PostgreSQL database schema + Alembic migrations for: leagues, teams, players, rosters, player_stats, projections, matchups
- Data import script: load existing `data/` JSON files into PostgreSQL
- Yahoo OAuth service class (wrap yfpy for backend use)
- Basic CRUD endpoints: teams, players, matchups, standings
- Refactor cron scripts to write to DB instead of (or in addition to) JSON files
- Docker Compose: FastAPI + PostgreSQL containers
- Health check and OpenAPI docs

### Phase 2: Frontend

Replace the Streamlit dashboard with a Next.js React application.

- Next.js project with TypeScript + Tailwind CSS
- API client layer (React Query + typed fetch wrappers)
- Dashboard pages:
  - Overview (league standings, today's top performers, matchup status)
  - Team detail (roster, daily/weekly performance, bench efficiency)
  - Player analysis (stats, projections vs actuals, trends)
  - Matchup view (head-to-head, category breakdown, win probability)
  - Recaps (display LLM-generated recaps, browsable by date)
- Interactive charts (Recharts or similar)
- Mobile-responsive design
- Docker container for frontend

### Phase 3: Intelligence API

Expose the enrichment and LLM capabilities as API endpoints.

- `/api/v1/analysis/recap/{date}` — Serve or generate daily recap
- `/api/v1/analysis/enriched/{date}` — Serve enriched daily data
- `/api/v1/analysis/missed-opportunities/{team_id}` — Team-specific missed opportunity history
- `/api/v1/analysis/projections-vs-actual` — Projection accuracy over time
- LLM service abstraction (Claude-only for now, structured for future providers)
- Prompt template management (load from `llm/` directory)
- Response caching for expensive LLM calls

### Phase 4: Advanced Features & Deployment

- Multi-league support (remove hardcoded league ID, support league selection)
- Roster optimization recommendations (LLM-powered)
- Trade analysis tools
- Waiver wire suggestions
- Docker-based deployment to VPS
- Nginx reverse proxy + SSL
- Automated backups for PostgreSQL

## AI safety

- `scripts/safe-git.sh` blocks `gh pr merge` when running under AI agents
- Never grant `--dangerously-skip-permissions` in CI/CD
- Review all AI-generated code before committing
- Data files committed by GitHub Actions bots go to main directly; code changes always go through PRs

## Key references

- Yahoo Fantasy Sports API: https://developer.yahoo.com/fantasysports/guide/
- yfpy library: https://github.com/uberfastman/yfpy
- yfpy docs: https://yfpy.uberfastman.com/
- FastAPI: https://fastapi.tiangolo.com/
- Next.js: https://nextjs.org/docs

---
> Source: [onurburak9/opencommish](https://github.com/onurburak9/opencommish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
