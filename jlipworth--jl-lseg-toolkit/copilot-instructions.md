## jl-lseg-toolkit

> Project context for Claude Code.

# CLAUDE.md

Project context for Claude Code.

## Project

**Package**: `jl-lseg-toolkit` - Python toolkit for LSEG financial data API.

**CLI Tools**: `lseg-earnings`, `lseg-screener`, `lseg-extract`, `lseg-scheduler`

## Quick Commands

```bash
uv sync
uv run pytest tests/ --no-cov
uv run pre-commit run --all-files
```

## Key References

| Topic | File |
|-------|------|
| Quick start & CLI usage | [README.md](README.md) |
| Setup (all platforms) | [docs/GETTING_STARTED.md](docs/GETTING_STARTED.md) |
| Timeseries module | [docs/TIMESERIES.md](docs/TIMESERIES.md) |
| System architecture | [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) |
| API patterns & fields | [docs/LSEG_API_REFERENCE.md](docs/LSEG_API_REFERENCE.md) |
| Development & testing | [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md) |
| Examples | [examples/](examples/) |

## Repo-local Claude config

- Agents: `.claude/agents/`
- Skills: `.claude/skills/` (e.g. `.claude/skills/timescaledb-access/SKILL.md`)
- Slash commands: `.claude/commands/` (e.g. `/timescaledb-access`)

TimescaleDB access workflow lives at `.claude/skills/timescaledb-access/SKILL.md`. Keep DB access agnostic: prefer env-driven config (`TSDB_*` or `POSTGRES_*`) and user-owned credentials/secret-manager setup. Codex users can symlink or copy `.claude/skills/timescaledb-access` into `$CODEX_HOME/skills/timescaledb-access` for direct skill pickup.

## Rules

- Exploratory scripts → `dev_scripts/` (gitignored)
- LSEG Workspace must be running on `localhost:9000`

---
> Source: [jlipworth/jl-lseg-toolkit](https://github.com/jlipworth/jl-lseg-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
