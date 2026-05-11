## behavior-sense

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BehaviorSense is a real-time user behavior stream analytics engine built with Python 3.11+ using a monorepo architecture. It processes user behavior events with sub-second latency, applies business rules, and generates user insights/tags.

## Commands

### Package Management (uv)

```bash
uv sync                           # Install all dependencies
uv add httpx                      # Add dependency to root
uv add --package behavior-audit httpx  # Add to specific package
uv run <command>                  # Run any command with virtual environment
```

### Development

```bash
# Run services
uv run uvicorn behavior_mock.main:app --port 8001
uv run uvicorn behavior_rules.main:app --port 8002
uv run uvicorn behavior_insight.main:app --port 8003
uv run uvicorn behavior_audit.main:app --port 8004
uv run python -m behavior_stream  # Faust stream processor

# Code quality
uv run ruff check libs/ packages/
uv run ruff format libs/ packages/
uv run mypy libs/core/src/behavior_core --ignore-missing-imports
uv run mypy packages/*/src --ignore-missing-imports
```

### Testing

```bash
# Fast tests (no external dependencies)
uv run pytest tests/test_api/test_mock_api.py tests/test_api/test_rules_api.py tests/test_integration/test_basic_integration.py -v

# Full tests with real dependencies (requires Docker)
TEST_REAL_DEPS=1 uv run pytest tests/ --cov=libs --cov=packages -v

# Or use the scripts
./scripts/run_tests.sh           # Mock mode
./scripts/run_tests.sh --real    # Real dependencies
```

## Architecture

### Data Flow

```
Mock (port 8001) → Pulsar (port 6650) → Stream (Faust) → Rules (port 8002) → Insight (port 8003)
                                                              ↓
                                                         Audit (port 8004)
```

### Monorepo Structure

```
libs/core/           # Shared library: config, models, security, middleware, utils
packages/mock/       # Event generator (FastAPI)
packages/stream/     # Real-time stream processing (Faust)
packages/rules/      # Rule engine API (FastAPI)
packages/insight/    # User insight/tagging API (FastAPI)
packages/audit/      # Manual review workflow (FastAPI)
apps/web/            # Frontend (Next.js, reserved)
tests/               # test_api/, test_integration/, test_core/, etc.
```

### Module Responsibilities

| Module | Tech | Port | Purpose |
|--------|------|------|---------|
| mock | FastAPI | 8001 | Generate test user behavior events |
| stream | Faust | - | Real-time event processing, aggregation, pattern detection |
| rules | FastAPI | 8002 | Rule matching engine with hot-reload |
| insight | FastAPI | 8003 | User profiling and tag management |
| audit | FastAPI | 8004 | Human-in-the-loop review workflow |

### Shared Library (libs/core)

- `config/` - Settings using pydantic-settings
- `models/` - Pydantic v2 data models
- `security/` - JWT auth, password hashing
- `middleware/` - Rate limiting, request tracing
- `utils/` - Logging (structlog), datetime utilities

## Code Conventions

### Commit Messages

All commits must follow [Conventional Commits](https://www.conventionalcommits.org/) in **English**:

```
feat(audit): add audit state machine for review workflow
fix(rules): prevent eval injection with AST parser
docs(api): update endpoint documentation
test(core): add unit tests for models
refactor: migrate to monorepo structure
```

### Code Style

- Line length: 100 characters
- Linter: ruff with rules E, F, I, N, W, UP, B, C4
- Type hints: Required for `behavior_core.*` (strict mode), optional for packages

### Python Best Practices

- Use dependency injection for database sessions in FastAPI
- Use `asyncio.gather()` for concurrent operations
- Use `orjson` for JSON serialization (3-5x faster than stdlib json)
- Use SQLAlchemy `selectinload` to avoid N+1 queries
- Set `expire_on_commit=False` for async sessions

## Technology Stack

| Layer | Technology |
|-------|------------|
| Runtime | Python 3.11+ |
| Package Manager | uv |
| Web Framework | FastAPI |
| Validation | Pydantic v2 |
| Stream Processing | Faust |
| Message Queue | Apache Pulsar |
| Database | PostgreSQL + SQLAlchemy async |
| Cache | Redis |
| Analytics | ClickHouse |
| Logging | structlog |

## Key Files

- `pyproject.toml` - Root config, uv workspace, ruff/mypy settings
- `wiki/architecture.md` - System architecture diagrams
- `wiki/best-practices.md` - FastAPI, Pydantic, SQLAlchemy, Faust best practices
- `tests/test_api/TEST_REPORT.md` - Test documentation

## CI/CD

The project uses GitHub Actions (`.github/workflows/ci.yml`) with:
1. **lint**: ruff check + mypy
2. **test-mock**: Fast tests without external dependencies
3. **test-integration**: Full tests with PostgreSQL and Redis containers
4. **build**: Docker images for each service (on main/master only)

## Infrastructure

- Docker Compose: `docker-compose.yml` and `docker-compose.test.yml`
- Dockerfile: `infrastructure/docker/Dockerfile` (multi-service build)

---
> Source: [afine907/behavior-sense](https://github.com/afine907/behavior-sense) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
