## process-miner

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A full-stack **Process Mining SaaS Platform** built with:
- **Backend**: FastAPI (Python 3.10+) with PM4Py for process mining algorithms
- **Frontend**: React 19 with Rspack bundling
- **Database**: SQLite (async via aiosqlite) with Alembic migrations
- **Data Processing**: DuckDB for high-performance data ingestion and analytics
- **Storage**: S3/MinIO for event log files (local filesystem in dev)
- **Workflow Orchestration**: Temporal for async long-running operations
- **Authentication**: JWT with access/refresh tokens

## Essential Commands

### Backend Development

```bash
# Start backend dev server (auto-reload on port 8001)
cd backend/src && ../.venv/bin/python -m uvicorn api.main:app --reload --port 8001

# Alternative using Makefile
cd backend && make dev

# Run all quality checks (lint + typecheck + security)
make check

# Run tests
make test
cd backend && .venv/bin/python -m pytest tests/ -v

# Run single test file
cd backend && .venv/bin/python -m pytest tests/api/test_datasets.py -v

# Database migrations
cd backend && .venv/bin/alembic upgrade head        # Apply migrations
cd backend && .venv/bin/alembic revision --autogenerate -m "description"  # Create migration
cd backend && .venv/bin/alembic current             # Check current version
cd backend && .venv/bin/alembic history             # View migration history
```

### Frontend Development

```bash
# Start frontend dev server (port 4200)
cd frontend-new && npm run start

# Build for production
cd frontend-new && npm run build

# Run tests
cd frontend-new && npm run test

# Regenerate API client from OpenAPI spec
cd frontend-new && npm run generate:sdk
```

### Full Stack Development

```bash
# Install all dependencies
make install

# Run full CI suite (lint + typecheck + security + tests)
make all

# Run both backend and frontend
make dev-full
```

## Architecture

### Backend: Layered Architecture with Feature Modules

The backend follows a **layered architecture** with strict dependency rules enforced by `import-linter`:

```
┌─────────────────────────────────────────────────────┐
│  API Layer (src/api/)                               │
│  - FastAPI app factory, middleware, exception handlers│
│  - Imports routers from features and infra          │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  Features Layer (src/features/)                     │
│  - Process mining algorithms and business logic     │
│  - Each feature: router.py, service.py, schemas.py  │
│  - Depends on: infra, shared                        │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  Infrastructure Layer (src/infra/)                  │
│  - Platform services: auth, organizations, projects │
│  - Core: config, database, logging, middleware      │
│  - Temporal workflows, jobs, health checks          │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  Shared Layer (src/shared/)                         │
│  - Utilities, types, constants                      │
│  - No dependencies on other layers                  │
└─────────────────────────────────────────────────────┘
```

**Key Principles:**
- **Features Independence**: Feature modules cannot depend on API layer
- **Infra Independence**: Infrastructure should not import from features
- **Shared Isolation**: Shared layer has no internal dependencies

### Multi-Tenant Authorization Hierarchy

```
Organization (owner)
  └── Workspace (owner/admin/editor/viewer)
        └── Project (inherits workspace permissions)
              └── Dataset (inherits project permissions)
```

### Dataset Lifecycle

```
PENDING → UPLOADING → UPLOADED → MAPPED → INGESTING → READY
                                                 ↓
                                            FAILED/ERROR
```

### Process Mining Capabilities

- **Discovery**: Alpha, Inductive, Heuristics, Split miners
- **Conformance**: Token replay, alignments, footprint comparison
- **Analytics**: Bottlenecks, cycle times, throughput, rework detection
- **Visualization**: DFG (Directly-Follows Graph), Petri nets
- **Predictions**: ML-based next activity and remaining time
- **Organizational**: Social network analysis, resource handovers
- **Simulation**: What-if analysis
- **OCPM**: Object-Centric Process Mining (OCEL 2.0)

### Backend Code Organization

```
backend/src/
├── api/                      # FastAPI application entry point
│   ├── main.py              # App factory, middleware, exception handlers
│   └── routers/             # Router imports and registration
├── application/              # CQRS application layer
│   ├── commands/            # Command handlers
│   ├── queries/             # Query handlers
│   └── projections/         # Read model projections
├── features/                 # Process mining feature modules
│   └── process_mining/
│       ├── models/          # SQLAlchemy models (Dataset, etc.)
│       ├── schemas/         # Pydantic schemas for API
│       ├── datasets/        # Dataset management (upload, ingest, CRUD)
│       ├── ingestion/       # DuckDB-based data ingestion
│       ├── discovery/       # Process discovery algorithms
│       ├── conformance/     # Conformance checking
│       ├── analytics/       # Performance analytics
│       ├── visualization/   # Graph visualization (DFG, Petri)
│       ├── predictions/     # ML predictions
│       ├── organizational/  # Organizational mining
│       ├── simulation/      # What-if simulation
│       ├── ocpm/            # Object-Centric PM (OCEL)
│       ├── ai/              # AI chat assistant
│       └── statistics/      # Dataset statistics
├── infra/                    # Infrastructure layer
│   ├── core/                # Config, logging, middleware, exceptions
│   ├── infrastructure/      # Database, storage, cache
│   ├── auth/                # JWT authentication
│   ├── users/               # User management + API routers
│   ├── organizations/       # Organization management
│   ├── workspaces/          # Workspace RBAC
│   ├── projects/            # Project management
│   ├── temporal/            # Temporal workflows and activities
│   ├── jobs/                # Background job tracking
│   ├── health/              # Health check endpoints
│   ├── audit/               # Audit logging
│   └── devconsole/          # Developer tools (SSE logs)
└── shared/                   # Shared utilities, types, constants
```

### Frontend: React 19 with Feature-Based Structure

```
frontend-new/
├── src/
│   ├── App.tsx              # App shell, providers, routing
│   ├── routes.tsx           # Explicit route definitions (all routes here)
│   ├── navigation.ts        # Navigation config and route mappings
│   ├── main.tsx             # Application entry point
│   ├── features/            # Feature modules
│   │   ├── platform/        # Workspace, projects, settings
│   │   ├── explorer/        # Process visualization (DFG, variants)
│   │   ├── analytics/       # Performance analytics dashboards
│   │   ├── discovery/       # Process discovery UI
│   │   ├── ai/              # AI predictions and chat
│   │   └── kpi/             # KPI dashboards
│   ├── shared/              # Shared code
│   │   ├── design-system.ts # UI component exports (use this!)
│   │   ├── context/         # UserContext, NotificationContext
│   │   ├── hooks/           # Shared custom hooks
│   │   ├── lib/             # Utilities (logger, formatters)
│   │   └── ui/              # DevConsole, ErrorBoundary
│   ├── api/                 # API integration
│   │   └── hooks/           # TanStack Query hooks
│   └── stores/              # Global state stores
└── libs/
    └── openapi-sdk/         # Auto-generated TypeScript API client
```

**Frontend Tech Stack:**
- React 19 with React Router v6
- Ant Design for UI components (import via `@/shared/design-system`)
- TanStack Query for server state management
- Cytoscape for process graph visualization
- Rspack for fast bundling

## Database Schema

**Key Tables:**
- `organizations`: Multi-tenant root entities
- `workspaces`: Collaborative work contexts
- `workspace_members`: Many-to-many with RBAC roles
- `projects`: Organizational containers
- `datasets`: Event log metadata and status
- `dataset_columns`: Detected schema from uploads
- `dataset_column_mappings`: User-defined process semantics
- `process_cases`: Trace/case representations
- `process_events`: Individual events in traces
- `process_models`: Discovered models (Petri nets, BPMN)
- `analyses`: Stored analysis results
- `jobs`: Async background job tracking

**Migration Workflow:**
```bash
# Create new migration after modifying models
cd backend && .venv/bin/alembic revision --autogenerate -m "add new field"

# Review generated migration in alembic/versions/
# Apply migration
cd backend && .venv/bin/alembic upgrade head

# Rollback one migration
cd backend && .venv/bin/alembic downgrade -1
```

## Testing Strategy

### Backend Tests (`backend/tests/`)
- **Framework**: pytest with pytest-asyncio for async support
- **Test Organization**: Mirrors `src/` structure
- **Fixtures**: `conftest.py` files provide shared test fixtures
- **Coverage**: Use `make test-cov` for coverage reports

**Test Categories:**
```python
tests/
├── api/              # API endpoint tests (integration)
├── domain/           # Domain logic tests (unit)
├── services/         # Service layer tests (unit + integration)
└── conftest.py       # Global fixtures (test client, test DB)
```

**Running Tests:**
```bash
# All tests
cd backend && .venv/bin/python -m pytest tests/ -v

# Specific test file
cd backend && .venv/bin/python -m pytest tests/api/test_datasets.py -v

# Specific test function
cd backend && .venv/bin/python -m pytest tests/api/test_datasets.py::test_upload_dataset -v

# With coverage
cd backend && .venv/bin/python -m pytest tests/ --cov=src --cov-report=html
```

### Frontend Tests (`frontend-new/`)
- **Framework**: Jest with React Testing Library
- **Run**: `cd frontend-new && npm run test`

## API Documentation

- **Swagger UI**: http://localhost:8001/docs (interactive API testing)
- **ReDoc**: http://localhost:8001/redoc (clean documentation)
- **OpenAPI Spec**: http://localhost:8001/openapi.json

**Dev Credentials:**
```
Email:    analyst@example.com
Password: TestPass123
```

**Key API Groups:**

| Group | Prefix | Description |
|-------|--------|-------------|
| Auth | `/api/v1/auth` | Login, refresh, logout |
| Datasets | `/api/v1/datasets` | Upload, ingest, CRUD |
| Discovery | `/api/v1/discovery` | Process model discovery |
| Conformance | `/api/v1/conformance` | Conformance checking |
| Analytics | `/api/v1/analytics` | Performance analytics |
| Visualization | `/api/v1/visualization` | DFG, Petri net graphs |
| Operations | `/api/v1/operations` | Long-running task status (SSE) |
| Health | `/health` | Liveness and readiness probes |

**User Flow Example:**
```
1. POST /api/v1/datasets/presign     → Get S3 upload URL
2. PUT  <upload_url>                 → Upload file
3. POST /api/v1/datasets/{id}/uploaded → Confirm, get columns
4. POST /api/v1/datasets/{id}/mapping  → Map case_id, activity, timestamp
5. POST /api/v1/datasets/{id}/ingest   → Start ingestion (returns workflow_id)
6. GET  /api/v1/operations/{id}/stream → SSE progress events
7. GET  /api/v1/datasets/{id}/statistics → View when READY
```

## Code Quality Standards

### Python (Backend)

**Linting & Formatting:**
- **Ruff**: Fast linter and formatter (replaces Black, isort, flake8)
- **Configuration**: `backend/pyproject.toml`
- **Auto-fix**: `cd backend && .venv/bin/python -m ruff check src/ tests/ --fix`

**Type Checking:**
- **mypy**: Static type checker
- **Run**: `cd backend && .venv/bin/python -m mypy src/`
- **Note**: Process mining feature modules have relaxed rules due to PM4Py's dynamic nature

**Security:**
- **Bandit**: Security vulnerability scanner
- **Run**: `cd backend && .venv/bin/python -m bandit -r src/`

**Import Linting:**
- **import-linter**: Enforces architectural boundaries
- **Run**: `cd backend && .venv/bin/lint-imports`
- **Config**: `[tool.importlinter]` in `backend/pyproject.toml`

### TypeScript (Frontend)

**Linting:**
- **ESLint**: TypeScript and React linting
- **Run**: `cd frontend-new && npx nx lint`

**Type Checking:**
- **TypeScript**: Strict type checking
- **Run**: `cd frontend-new && npx nx typecheck`

## Development Workflows

### Adding a New API Endpoint

1. Define schema in `src/features/process_mining/schemas/`
2. Add model if needed in `src/features/process_mining/models/`
3. Create service logic in `src/features/process_mining/{feature}/service.py`
4. Add router endpoint in `src/features/process_mining/{feature}/router.py`
5. Register router in `src/api/routers/__init__.py`
6. Import router in `src/api/main.py`
7. Write tests in `backend/tests/api/`
8. Regenerate frontend SDK: `cd frontend-new && npm run generate:sdk`

### Adding a New Process Mining Algorithm

1. Implement in `src/features/process_mining/{category}/service.py`
2. Add schemas to `src/features/process_mining/schemas/{category}.py`
3. Create router in `src/features/process_mining/{category}/router.py`
4. Register in `src/api/routers/__init__.py` and `src/api/main.py`
5. Add tests in `backend/tests/api/`

### Modifying Database Schema

1. Update SQLAlchemy models in `src/features/process_mining/models/` or `src/infra/models.py`
2. Create migration: `cd backend && .venv/bin/alembic revision --autogenerate -m "description"`
3. Review and edit generated migration in `alembic/versions/`
4. Test migration: `alembic upgrade head` and `alembic downgrade -1`
5. Commit migration file with code changes

## Key Dependencies

### Backend
- **fastapi**: Web framework with async support
- **pm4py**: Process mining algorithms (discovery, conformance, analytics)
- **sqlalchemy**: Async ORM with SQLite/aiosqlite
- **pydantic**: Data validation and serialization
- **duckdb**: High-performance data ingestion and analytics
- **temporalio**: Workflow orchestration for long-running operations
- **structlog**: Structured JSON logging
- **slowapi**: Rate limiting

### Frontend
- **react**: UI framework (v19)
- **@tanstack/react-query**: Server state management
- **antd**: UI component library
- **cytoscape**: Process graph visualization
- **react-router-dom**: Client-side routing
- **rspack**: Fast bundling

## Environment Setup

### First-Time Setup

```bash
# Backend
cd backend
python3 -m venv .venv
source .venv/bin/activate  # or `.venv/bin/activate` on macOS/Linux
pip install -e ".[dev]"
alembic upgrade head

# Frontend
cd frontend-new
npm install

# Start both (two terminals)
# Terminal 1: Backend
cd backend/src && ../.venv/bin/python -m uvicorn api.main:app --reload --port 8001

# Terminal 2: Frontend
cd frontend-new && npm run start
```

### MVP Development Data

On startup, the backend automatically seeds:
- **Organization**: `mvp-org-001` (Demo Organization)
- **Workspace**: `mvp-ws-001` (Default Workspace)
- **User**: `mvp-user-001` (analyst@company.local)

Frontend hardcodes these IDs for rapid development.

## Important Patterns

### Long-Running Operations with Temporal

Long-running operations (data ingestion, discovery) use Temporal workflows:

```python
# Starting a workflow returns a workflow_id for tracking
workflow_id = await start_ingestion_workflow(dataset_id)

# Frontend subscribes to progress via SSE
# GET /api/v1/operations/{workflow_id}/stream
```

**SSE Events:**
- `workflow:progress` - Overall progress update
- `step:started`, `step:progress`, `step:completed` - Step-level updates
- `workflow:completed`, `workflow:failed` - Terminal states

### Event Log Loading Pattern

```python
# Standard pattern for accessing event logs
from src.features.process_mining.ingestion import load_event_log

event_log = await load_event_log(dataset_id)  # Returns PM4Py EventLog
```

### Error Handling (RFC 7807)

Backend uses **RFC 7807 Problem Details** for consistent error responses:

```python
from src.infra.core.exceptions import AppException, ErrorCode

raise AppException(
    message="Dataset not found",
    error_code=ErrorCode.RESOURCE_NOT_FOUND,
    status_code=404,
    details={"dataset_id": dataset_id}
)
```

**Response Format:**
```json
{
  "type": "about:blank",
  "title": "Not Found",
  "status": 404,
  "detail": "Dataset with id 'xyz' not found",
  "error_code": "RESOURCE_NOT_FOUND",
  "correlation_id": "req-abc123"
}
```

### Authentication Pattern

```python
from src.infra.auth import get_current_user

@router.get("/protected")
async def protected_endpoint(user: User = Depends(get_current_user)):
    # user is authenticated
    pass
```

## Debugging

### Backend Logs
- **Structured logging**: All logs use structlog with JSON output in production
- **Dev Console**: http://localhost:8001/api/v1/dev/logs/stream for real-time logs (debug mode only)

### Frontend
- **React DevTools**: Browser extension for component inspection
- **Network Tab**: Inspect API calls (CORS configured for localhost:4200)

### Database Inspection
```bash
# SQLite CLI
sqlite3 backend/data/db/process_mining.db

# Check tables
.tables

# Query data
SELECT * FROM datasets;
```

## Performance Considerations

- **DuckDB**: Used for large-scale data ingestion (vectorized processing, columnar storage)
- **Temporal Workflows**: Offload heavy computations (ingestion, discovery, analytics)
- **Lazy Loading**: Frontend routes use React.lazy() for code splitting
- **Pagination**: All list endpoints support pagination
- **Rate Limiting**: 100 req/min standard, 10 req/min for uploads

## Git Workflow

The repository uses feature branches and merge commits:

```bash
# Current branch
git branch  # You're on: polish-refactor-fixes

# Main branch
git checkout main  # Main branch for PRs
```

When creating PRs, target the `main` branch.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/namanag97) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
