## message-database

> Hostaway Message Database — a Python ETL (Extract, Transform, Load) pipeline that retrieves messages from the Hostaway API (OAuth 2.0), transforms them, and stores them in MongoDB. Supports incremental daily updates via cron scheduling, dry-run mode, and Docker deployment.

# CLAUDE.md

## Project Overview

Hostaway Message Database — a Python ETL (Extract, Transform, Load) pipeline that retrieves messages from the Hostaway API (OAuth 2.0), transforms them, and stores them in MongoDB. Supports incremental daily updates via cron scheduling, dry-run mode, and Docker deployment.

## Repository Structure

```
message-database/               # Git root
├── CLAUDE.md                   # This file
├── README.md                   # Project documentation
└── message-database/           # Project directory (all code lives here)
    ├── src/
    │   ├── message_database/   # Main Python package (canonical)
    │   │   ├── api/
    │   │   │   └── hostaway_client.py   # Hostaway API client (OAuth, pagination, retry)
    │   │   ├── database/
    │   │   │   └── mongodb.py           # MongoDB adapter (connect, upsert, indexes)
    │   │   ├── models/
    │   │   │   └── message.py           # Pydantic data models (Message, Property, Guest, Reservation)
    │   │   ├── pipeline/
    │   │   │   └── etl.py               # ETL orchestration (extract, transform, load)
    │   │   ├── scheduler/
    │   │   │   └── daily_job.py         # Cron job runner
    │   │   ├── utils/
    │   │   │   └── logger.py            # Logging setup + email notification stubs
    │   │   ├── config.py                # Environment-based configuration
    │   │   └── main.py                  # CLI entry point (argparse)
    │   └── (legacy mirrors at src/ top level — prefer src/message_database/)
    ├── tests/                   # Pytest test suite
    │   ├── test_etl.py          # ETL pipeline tests
    │   ├── test_api_client.py   # API client unit tests
    │   ├── test_api.py          # Additional API tests
    │   ├── test_api_params.py   # API parameter validation tests
    │   ├── test_mongodb.py      # MongoDB operation tests
    │   └── test_yesterday.py    # Yesterday message retrieval tests
    ├── scripts/                 # Shell/Python convenience scripts
    │   ├── run_etl_yesterday.sh
    │   ├── run_tests.sh
    │   ├── run_yesterday_test.sh
    │   └── daily_job.py
    ├── docs/                    # Detailed documentation
    │   ├── architecture.md
    │   ├── authentication.md
    │   ├── deployment.md
    │   ├── etl_pipeline.md
    │   └── project_refactoring.md
    ├── Dockerfile
    ├── requirements.txt
    ├── setup.py
    └── pytest.ini
```

**Important:** The `src/` directory contains both top-level modules (legacy) and the `message_database/` package. The canonical source is `src/message_database/` — this is what `setup.py` packages and what imports reference via relative imports.

## Tech Stack

- **Language:** Python 3.8+
- **Data validation:** Pydantic 2.x
- **HTTP client:** requests
- **Database:** MongoDB 4.4+ via pymongo (with SRV support)
- **Configuration:** python-dotenv (`.env` files)
- **Testing:** pytest + pytest-mock
- **Containerization:** Docker (python:3.9-slim base)

## Common Commands

All commands should be run from the `message-database/message-database/` directory unless noted.

### Install dependencies

```bash
pip install -r requirements.txt
```

### Run the ETL process

```bash
python -m message_database.main etl                              # Full ETL
python -m message_database.main etl --since 2023-01-01T00:00:00  # Since timestamp
python -m message_database.main etl --days 7                     # Last N days
```

### Run tests

```bash
cd message-database/message-database && pytest
```

pytest is configured in `pytest.ini`:
- Test discovery: `tests/test_*.py`
- Coverage: `--cov=src --cov-report=term-missing`
- Verbose output enabled by default (`-v`)

### Run tests via convenience script

```bash
./scripts/run_tests.sh --client-id YOUR_ID --client-secret YOUR_SECRET
```

### Docker build and run

```bash
docker build -t message-database .
docker run --env-file .env message-database
```

## Architecture

### Data Flow

```
Hostaway API  →  Extract (HostawayClient)  →  Transform (Pydantic models)  →  Load (MongoDB)
                 - OAuth 2.0 auth              - Validation                    - Upsert by message_id
                 - Pagination                   - Type coercion                - Auto-indexing
                 - Rate limiting                - Normalization
                 - Retry with backoff
```

### Key Components

| Component | File | Responsibility |
|-----------|------|----------------|
| API Client | `api/hostaway_client.py` | OAuth token management, paginated API requests, retry logic |
| Database | `database/mongodb.py` | MongoDB connection, upsert operations, index management |
| Models | `models/message.py` | Pydantic models for Message, Property, Guest, Reservation |
| Pipeline | `pipeline/etl.py` | ETL orchestration, per-message error handling |
| Config | `config.py` | Environment variable loading and validation |
| CLI | `main.py` | Argparse-based command-line interface |
| Scheduler | `scheduler/daily_job.py` | Daily cron job execution |
| Logger | `utils/logger.py` | Dual console+file logging, email notification stubs |

### Module Import Pattern

All internal imports use relative imports from `message_database` package:
```python
from ..api.hostaway_client import api_client, HostawayAPIError
from ..database.mongodb import db
from ..models.message import Message, Property, Guest, Reservation
from ..config import validate_config
```

### Singleton Instances

- `api_client` — global `HostawayClient` instance in `api/hostaway_client.py`
- `db` — global `MongoDB` instance in `database/mongodb.py`
- `pipeline` — global `ETLPipeline` instance in `pipeline/etl.py`

## Configuration

All configuration is via environment variables, loaded from a `.env` file by `python-dotenv`.

### Required Variables

| Variable | Description |
|----------|-------------|
| `HOSTAWAY_CLIENT_ID` | OAuth 2.0 client ID |
| `HOSTAWAY_CLIENT_SECRET` | OAuth 2.0 client secret |
| `MONGODB_URI` | MongoDB connection string |

### Optional Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `HOSTAWAY_BASE_URL` | `https://api.hostaway.com/v1` | API base URL |
| `MONGODB_DATABASE` | `hostaway_messages` | Database name |
| `MONGODB_COLLECTION` | `messages` | Collection name |
| `LOG_LEVEL` | `INFO` | Logging level (DEBUG/INFO/WARNING/ERROR/CRITICAL) |
| `LOG_DIR` | `./logs` | Log file directory |
| `API_REQUEST_DELAY` | `1.0` | Delay between API requests (seconds) |
| `ENABLE_DRY_RUN` | `false` | Skip actual API calls and DB writes |
| `NOTIFICATION_EMAIL` | (empty) | Email for error notifications |

## Coding Conventions

- **Python version:** 3.8+ (no walrus operator in older code, but f-strings used throughout)
- **Data models:** Pydantic `BaseModel` with field validators for data integrity
- **Error handling:** Per-message try/except in ETL loop; pipeline continues on individual failures
- **Custom exceptions:** `HostawayAPIError` for API-specific errors
- **Retry logic:** Exponential backoff for API calls (max 3 attempts) and MongoDB connections (max 3 attempts, 5-sec delay)
- **Logging:** Use the shared `logger` from `utils/logger.py`; daily rotating log files
- **Naming:** snake_case for functions/variables, PascalCase for classes
- **Docstrings:** Module-level and class/method docstrings present throughout

## Testing Conventions

- All test files in `tests/` follow `test_*.py` naming
- Test classes follow `Test*` naming, test functions follow `test_*` naming
- External dependencies (API, MongoDB) are mocked using `pytest-mock` fixtures
- Tests cover both success and error paths
- No `.env` file needed for unit tests (mocks handle configuration)

## Things to Watch Out For

1. **Dual source layout:** `src/` contains both legacy top-level modules and the canonical `src/message_database/` package. Only edit files in `src/message_database/`.
2. **No CI/CD pipeline:** There are no GitHub Actions or other CI configs. Tests must be run manually.
3. **Email notifications:** The `send_error_notification()` function in `utils/logger.py` is a stub — SMTP is not configured.
4. **Sensitive files:** `.env` is gitignored. Never commit API credentials or MongoDB URIs.
5. **Dockerfile CMD:** Uses `python -m src.main etl` which references the legacy path — may need updating if deploying via Docker.
6. **Coverage target:** Coverage is reported via `--cov=src` which covers both legacy and package code.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jobo0o)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Jobo0o)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
