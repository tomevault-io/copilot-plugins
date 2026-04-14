## htbase

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HTBase is a web archiving service that provides a REST API for saving web pages using multiple archival methods (monolith, SingleFile, screenshot, PDF, readability). It wraps archiving commands in `ht` for live terminal preview, stores metadata in PostgreSQL, and provides optional AI-powered summarization and entity extraction.

## Architecture

### Core Components

**FastAPI Server** (`app/server.py`)
- Main application entry point with lifespan context manager
- Registers archivers on `app.state.archivers` during startup
- Mounts API routes via `app/api/` and web UI routes via `app/web/`
- Serves archived files statically under `/files` mount

**Archivers** (`app/archivers/`)
- All archivers extend `BaseArchiver` and implement `archive(url, item_id) -> ArchiveResult`
- Registration order in server.py matters for "all" pipeline: readability runs first (DOM dump reused by monolith), then monolith/singlefile-cli, finally screenshot/pdf
- Each archiver saves to `{DATA_DIR}/{item_id}/{archiver_name}/output.{ext}`

**HTRunner** (`app/core/ht_runner.py`)
- Wraps shell commands in `ht` terminal for live preview at port 7681
- Uses stdin/stdout JSON communication protocol
- Thread-safe command serialization via `self.lock`
- All ht interactions should use `.send_command()` or `.send_command_and_snapshot()`

**Database** (`app/db/`)
- PostgreSQL backend (SQLAlchemy + psycopg driver)
- `archived_urls` table enforces URL uniqueness (single row per URL)
- `archive_artifact` tracks individual archiver runs (one row per archiver per URL)
- `url_metadata` stores readability-extracted metadata (title, byline, text, etc.)
- `article_summaries`, `article_entities`, `article_tags` for AI-powered semantic analysis
- Connection string built in `AppSettings.database_url` from DB_HOST/DB_PORT/DB_NAME/DB_USER/DB_PASSWORD env vars

**Task Managers** (`app/task_manager/`)
- `BackgroundTaskManager`: Generic queue-based background worker (ABC with `.process()` method)
- `ArchiverTaskManager`: Processes batch archiving tasks; supports requeue for failed/pending artifacts
- `SummarizationTaskManager`: Generates summaries/tags/entities using HuggingFace TGI backend
- Task managers are started during server lifespan and run in daemon threads

**Configuration** (`app/core/config.py`)
- Environment-based settings via pydantic-settings (loads from .env file)
- Key settings: DATA_DIR, DB_HOST, DB_PORT, DB_NAME, DB_USER, DB_PASSWORD, ENABLE_SUMMARIZATION, START_HT, SKIP_EXISTING_SAVES
- Access via `get_settings()` (lru_cached singleton)

**Frontend** (`frontend/`)
- React + TypeScript + Vite + TailwindCSS
- React Router for navigation, TanStack Query for data fetching
- Communicates with backend via axios

### Data Flow

1. POST `/save/{archiver}` receives SaveRequest (url, id)
2. URL reachability checked; 404s recorded immediately as failed
3. For each archiver: check if already saved (if SKIP_EXISTING_SAVES=true), else insert pending artifact
4. Archiver runs via ht_runner.send_command_and_snapshot()
5. Result recorded in archive_artifact table with success/exit_code/saved_path
6. If archiver is in SUMMARY_SOURCE_ARCHIVERS and ENABLE_SUMMARIZATION=true, enqueue summarization task
7. SummarizationTaskManager generates summary/tags/entities and stores in semantic tables

## Development Commands

### Docker

Build and start services:
```bash
docker compose up --build
```

Watch mode (rebuilds on file changes):
```bash
docker compose watch
```

View logs:
```bash
docker compose logs -f
```

Stop services:
```bash
docker compose down
```

### Testing

Run unit tests:
```bash
pytest tests/unit
```

Run integration tests (requires PostgreSQL):
```bash
# Set DB connection env vars first
export DB_HOST=127.0.0.1 DB_PORT=5432 DB_NAME=htbase DB_USER=postgres DB_PASSWORD=postgres
alembic upgrade head  # Run migrations
pytest tests/integration
```

Run with coverage:
```bash
pytest --cov=app --cov-report=html tests/
```

Run single test file:
```bash
pytest tests/unit/test_models.py -v
```

### Database Migrations

Create new migration:
```bash
alembic revision --autogenerate -m "description"
```

Apply migrations:
```bash
alembic upgrade head
```

Rollback one migration:
```bash
alembic downgrade -1
```

View migration history:
```bash
alembic history
```

**Note**: Database URL is constructed automatically from environment variables in `alembic/env.py` using AppSettings.database_url property.

### Frontend

Install dependencies:
```bash
cd frontend && npm install
```

Development server (with hot reload):
```bash
cd frontend && npm run dev
```

Build for production:
```bash
cd frontend && npm run build
```

Lint:
```bash
cd frontend && npm run lint
```

Format:
```bash
cd frontend && npm run format
```

### Manual Testing

Save a page via monolith:
```bash
curl -X POST http://localhost:8000/save/monolith \
  -H 'Content-Type: application/json' \
  -d '{"id":"test123","url":"https://example.com"}'
```

Save via all archivers:
```bash
curl -X POST http://localhost:8000/save/all \
  -H 'Content-Type: application/json' \
  -d '{"id":"test123","url":"https://example.com"}'
```

Batch create saves:
```bash
curl -X POST http://localhost:8000/tasks/batch-create \
  -H 'Content-Type: application/json' \
  -d '{"archiver":"all","items":[{"item_id":"id1","url":"https://example.com"}]}'
```

Requeue failed saves:
```bash
curl -X POST http://localhost:8000/tasks/requeue-failed \
  -H 'Content-Type: application/json' \
  -d '{"priorities":["singlefile-cli","monolith"]}'
```

## Key Implementation Patterns

### Adding a New Archiver

1. Create new class in `app/archivers/` extending `BaseArchiver`
2. Implement `archive(url: str, item_id: str) -> ArchiveResult` method
3. Use `ht_runner.send_command_and_snapshot()` to run commands via ht
4. Save output to `{DATA_DIR}/{safe_item_id}/{self.name}/output.{ext}`
5. Register in `server.py` lifespan context: `app.state.archivers["{name}"] = YourArchiver(...)`

### Background Task Processing

Task managers extend `BackgroundTaskManager[TaskT]` and implement `.process(task: TaskT)`:
```python
class MyTaskManager(BackgroundTaskManager[MyTask]):
    def process(self, task: MyTask) -> None:
        # Handle task here
        pass

# Usage
manager = MyTaskManager()
manager.start()  # Starts daemon thread
manager.submit(MyTask(...))  # Adds to queue
```

### Database Operations

Use repository functions in `app/db/repository.py` for all DB access:
- `insert_pending_save()`: Create pending artifact before archiving
- `finalize_save_result()`: Update artifact after archiving completes
- `find_existing_success_save()`: Check if URL+archiver already saved
- `get_archived_url_by_id()`: Lookup by rowid
- `upsert_article_summary()`: Store summary data

### Working with HTRunner

All command execution should go through HTRunner:
```python
from core.ht_runner import HTRunner

ht_runner = request.app.state.ht_runner
result = ht_runner.send_command_and_snapshot(
    cmd_parts=["monolith", "--output", str(out_path), url],
    timeout_seconds=120
)
exit_code = result.exit_code
output_lines = result.combined_output  # list of strings
```

### Configuration Management

Never hardcode paths or connection strings. Use settings:
```python
from core.config import get_settings

settings = get_settings()
data_dir = settings.data_dir  # Path object
db_url = settings.database_url  # postgresql+psycopg://...
chromium_bin = settings.chromium_bin
```

### Summarization Integration

If adding features that generate content for summarization:
1. Check `settings.enable_summarization` before enqueuing
2. Use `SummarizationCoordinator.maybe_enqueue_summary()` to queue task
3. Ensure archiver name is in `settings.summary_source_archivers` list
4. Summary tasks are processed asynchronously by SummarizationTaskManager

## Important Constraints

- **URL Uniqueness**: `archived_urls.url` has unique constraint. Use `db.repository.get_or_create_archived_url()` to avoid conflicts
- **Artifact Uniqueness**: `(archived_url_id, archiver)` pair is unique in `archive_artifact` table
- **HT Serialization**: HTRunner uses a lock to serialize commands; do not bypass with direct subprocess calls
- **Database Driver**: Always use `postgresql+psycopg://` scheme (not `postgresql://` or `postgresql+psycopg2://`)
- **Safe Filenames**: Always call `sanitize_filename(item_id)` before constructing file paths
- **Environment Variables**: All configuration via environment; update `app/core/config.py` AppSettings class for new settings

## Testing Considerations

- Tests use `temp_env` fixture to override DATA_DIR and DB_PATH
- Set `START_HT=false` to disable ht runner in tests
- Integration tests require PostgreSQL service running and migrations applied
- Use `DummyArchiver` in tests to avoid external binary dependencies (see `tests/conftest.py`)
- Test client injects minimal archivers and task_manager on `app.state`

## Development Planning

**SUGGEST_REFACTOR.md** tracks planned refactoring work and technical debt. When working on the codebase:
- Review TODO.md for context on known issues and planned improvements
- Update TODO.md when discovering new refactoring opportunities
- Mark items complete when implementing suggested changes
- Keep TODO.md focused on technical architecture and code quality (not feature requests)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jayteealao) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
