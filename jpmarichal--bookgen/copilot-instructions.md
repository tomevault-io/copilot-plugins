## bookgen

> BookGen is a production-ready FastAPI application that automates biography generation. These rules help AI agents support development, maintenance, and bug fixes.


# BookGen Development Rules

## Overview

BookGen is a production-ready FastAPI application that automates biography generation. These rules help AI agents support development, maintenance, and bug fixes.

## Core Principles

1. **Minimal Changes**: Make the smallest possible changes to achieve the goal
2. **Test-Driven**: Run existing tests before and after changes
3. **Documentation**: Keep docs in sync with code changes
4. **Clean Root**: Avoid adding files to root directory

## Project Structure

### Main Application (`/src`)
```
src/
├── main.py                    # FastAPI entry point
├── worker.py                  # Celery worker configuration
├── api/                       # REST API endpoints
│   ├── routers/              # Endpoint routers
│   │   ├── biographies.py   # Biography generation endpoints
│   │   ├── sources.py       # Source validation endpoints
│   │   ├── metrics.py       # Metrics and monitoring
│   │   └── websocket.py     # Real-time updates
│   ├── models/              # Pydantic request/response models
│   └── middleware/          # Rate limiting, logging
├── services/                # Business logic services
│   ├── concatenation.py    # Biography file concatenation
│   ├── word_exporter.py    # Word document export
│   ├── source_validator.py # Source URL validation
│   ├── length_validator.py # Content length validation
│   ├── content_analyzer.py # Content quality analysis
│   ├── hybrid_generator.py # Hybrid generation strategy
│   └── source_generator.py # Automatic source generation
├── engine/                  # State machine engine
│   └── biography_engine.py # Generation workflow orchestration
├── tasks/                   # Celery async tasks
│   ├── biography_tasks.py  # Main generation tasks
│   ├── validation_tasks.py # Async validation
│   └── export_tasks.py     # Export tasks
├── strategies/              # Content generation strategies
│   ├── automatic.py        # Fully automatic generation
│   ├── hybrid.py          # Semi-automatic with validation
│   └── personalized.py    # Custom strategies per character
├── repositories/           # Database access layer
├── models/                 # Database models (SQLAlchemy)
└── config/                 # Configuration management
```

### Documentation (`/docs`)
```
docs/
├── INDEX.md               # Documentation navigation
├── getting-started/       # Installation and setup
├── api/                   # API documentation
├── user-guide/           # End user guides
├── architecture/         # System architecture
├── operations/           # Deployment and ops
├── technical/            # Technical deep-dives
│   ├── quickstart/      # Component quick starts
│   ├── components/      # Component documentation
│   ├── deployment/      # Deployment guides
│   └── testing/         # Testing strategies
├── archive/             # Historical documentation
└── project/             # Project planning (Spanish)
```

### Tests (`/tests`)
```
tests/
├── conftest.py           # Pytest fixtures
├── test_*.py            # Unit tests (mirror src/ structure)
└── integration/         # Integration tests
```

### Development (`/development`)
```
development/
├── scripts/             # Development utilities
│   ├── legacy/         # Legacy manual scripts (archived)
│   └── deploy-vps.sh   # VPS deployment script
└── docker/             # Docker configurations
```

### Infrastructure (`/infrastructure`)
```
infrastructure/
├── Dockerfile          # Production Docker image
├── docker-compose.yml  # Development environment
├── docker-compose.prod.yml  # Production environment
└── nginx/             # Nginx reverse proxy config
```

### Data Directories (gitignored)
```
bios/                   # Biography working directory
├── {character_name}/  # Per-character directory
│   ├── research/      # Research materials, sources, plans
│   ├── chapters/      # Chapter markdown files
│   ├── sections/      # Special sections (prologo, epilogo, etc.)
│   ├── output/        # Generated files
│   │   ├── markdown/  # Concatenated markdown
│   │   ├── word/      # Word documents
│   │   └── kdp/       # Amazon KDP assets
│   └── control/       # Quality control logs
```

## Quick Code Navigation

### Finding Components

**Biography Generation Flow:**
1. API endpoint: `src/api/routers/biographies.py` → `POST /biographies`
2. Celery task: `src/tasks/biography_tasks.py` → `generate_biography_task()`
3. State machine: `src/engine/biography_engine.py` → `BiographyEngine`
4. Strategies: `src/strategies/` (automatic, hybrid, personalized)
5. Services: `src/services/` (all business logic)

**Source Validation:**
1. API: `src/api/routers/sources.py`
2. Service: `src/services/source_validator.py`
3. Task: `src/tasks/validation_tasks.py`

**Export/Concatenation:**
1. Concatenation: `src/services/concatenation.py` → `ConcatenationService`
2. Word export: `src/services/word_exporter.py` → `WordExporter`
3. ZIP export: `src/services/zip_export_service.py`
4. Tasks: `src/tasks/export_tasks.py`

**Notifications:**
1. Service: `src/services/notifications.py`
2. WebSocket: `src/websocket/manager.py`
3. Email: `src/email/service.py`

### Configuration Files

- `.env` - Environment variables (gitignored)
- `.env.production.example` - Production config template
- `requirements.txt` - Python dependencies
- `pytest.ini` - Pytest configuration
- `alembic.ini` - Database migrations
- `docker-compose.yml` - Development Docker setup

## Common Tasks

### Running Tests
```bash
# All tests
pytest

# Specific test file
pytest tests/test_concatenation.py

# With coverage
pytest --cov=src

# Verbose
pytest -v
```

### Database Migrations
```bash
# Create migration
alembic revision --autogenerate -m "description"

# Apply migrations
alembic upgrade head

# Rollback
alembic downgrade -1
```

### Running the Application
```bash
# Development (Docker)
docker-compose up

# Production (Docker)
docker-compose -f docker-compose.prod.yml up -d

# Local development
uvicorn src.main:app --reload

# Celery worker
celery -A src.worker worker --loglevel=info
```

### Checking Code Quality
```bash
# Run linters (if configured)
# Note: Check if trunk, black, or other linters are set up
trunk check

# Format code
black src/ tests/
```

## Business Rules

### 1. Directory Organization

**Keep Root Clean:**
- Documentation goes in `/docs`, not root
- Scripts go in `/development/scripts`
- Temporary files go in `/tmp` (gitignored)
- Build artifacts are gitignored
- New markdown files should go in `/docs` subdirectories

**Existing Root Files (acceptable):**
- README.md - Main project readme
- DOCUMENTATION_MAP.md - Documentation index
- Requirements and config files (.env.example, requirements.txt, etc.)

**Don't Add to Root:**
- New .md files (use /docs)
- Test files (use /tests)
- Scripts (use /development/scripts)
- Data files (use /bios or gitignored directories)

### 2. Testing Standards

**Test Organization:**
- Unit tests: `/tests/test_*.py` (mirrors src/ structure)
- Integration tests: `/tests/integration/`
- Test fixtures: `/tests/conftest.py`
- Test data: `/tests/fixtures/`

**Test Requirements:**
- All new features must have tests
- Tests should be isolated and repeatable
- Use pytest fixtures for setup/teardown
- Mock external services (OpenRouter API, etc.)
- Don't commit test artifacts

### 3. Documentation Standards

**When to Update Docs:**
- API changes → Update `/docs/api/`
- Architecture changes → Update `/docs/architecture/`
- New features → Update `/docs/user-guide/`
- Deployment changes → Update `/docs/operations/`
- Component changes → Update `/docs/technical/components/`

**Documentation Format:**
- Use clear headings and structure
- Include code examples where relevant
- Link to related documentation
- Keep technical details in `/docs/technical/`
- Keep user-facing content in `/docs/user-guide/`

### 4. Code Quality Standards

**Python Code:**
- Follow PEP 8 style guide
- Use type hints where beneficial
- Write clear docstrings for public APIs
- Keep functions focused and small
- Use meaningful variable names

**API Design:**
- RESTful endpoint naming
- Consistent response formats
- Proper HTTP status codes
- Input validation with Pydantic
- Error handling with appropriate messages

**Services:**
- Single responsibility principle
- Dependency injection
- Clear interfaces
- Proper error handling
- Logging for debugging

## Bug Fixing Workflow

1. **Understand the Issue:**
   - Read the bug report carefully
   - Reproduce the issue if possible
   - Identify affected components

2. **Locate the Code:**
   - Use project structure above to find relevant files
   - Check related services and tasks
   - Review recent changes if applicable

3. **Make Minimal Changes:**
   - Fix only what's broken
   - Don't refactor unrelated code
   - Keep changes focused

4. **Test the Fix:**
   - Run existing tests: `pytest`
   - Add new tests if needed
   - Verify the fix manually

5. **Update Documentation:**
   - Update docs if behavior changed
   - Add comments for complex fixes
   - Update CHANGELOG if exists

## File Placement Guide

**Where to put new files:**

- **Python code:** `/src` in appropriate subdirectory
- **Tests:** `/tests` mirroring src structure
- **Documentation:** `/docs` in appropriate section
- **Scripts:** `/development/scripts`
- **Configuration:** Root (if global) or `/infrastructure`
- **Database migrations:** `/alembic/versions`
- **Temporary files:** `/tmp` (gitignored)

**Don't create:**
- Loose .md files in root
- Test files outside `/tests`
- Scripts in root
- Data files that should be gitignored

## Related Documentation

- [System Architecture](../../docs/architecture/system-overview.md)
- [API Documentation](../../docs/api/overview.md)
- [Deployment Guide](../../docs/operations/deployment.md)
- [Testing Strategy](../../docs/technical/testing/TESTING_STRATEGY.md)
- [Directory Migration](../../docs/technical/components/DIRECTORY_MIGRATION.md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JPMarichal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
