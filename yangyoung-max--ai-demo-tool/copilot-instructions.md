## ai-demo-tool

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Commands

### Development
```bash
make dev                 # Start both backend (port 8000) and frontend (port 5173)
make dev-backend         # Start FastAPI backend only
make dev-frontend        # Start React frontend only
make install             # Install all dependencies (backend + frontend)
make setup               # Initial setup: install + run migrations
```

### Testing & Code Quality
```bash
make test                # Run all tests
make test-backend        # Run backend pytest tests
make test-frontend       # Run frontend vitest tests
make lint                # Run all linters
make format              # Format all code
make check               # Run lint + tests (CI check)
```

### Database
```bash
make db-upgrade          # Apply database migrations
make db-migrate MESSAGE="description"  # Create new migration
make db-history          # View migration history
```

### Single Test Execution
```bash
# Backend
cd backend && uv run pytest tests/test_specific_file.py::test_function -v

# Frontend
cd frontend && npm test -- specific_test_file
```

## High-Level Architecture

This project uses a **3-tier architecture with Adapter pattern** for database abstraction:

```
API Layer (FastAPI routes)
    ↓
Service Layer (Facade pattern)
    ↓
Adapter Registry (Factory pattern)
    ↓
Database Adapters (PostgreSQL, MySQL, etc.)
```

### Key Architectural Patterns

1. **Adapter Pattern**: Each database type has a dedicated adapter implementing `DatabaseAdapter` ABC
   - `app/adapters/base.py` - Abstract base class with 7 required methods
   - `app/adapters/postgresql.py` - PostgreSQL implementation
   - `app/adapters/mysql.py` - MySQL implementation
   - Adding new database: Create adapter file, register in registry, update enum

2. **Facade Pattern**: `DatabaseService` provides high-level interface coordinating adapters, validators, and business logic
   - Located: `app/services/database_service.py`
   - Global instance: `database_service` used throughout API layer

3. **Factory Pattern**: `DatabaseAdapterRegistry` manages adapter lifecycle and connection pools
   - Located: `app/adapters/registry.py`
   - Global instance: `adapter_registry` initialized with all supported databases

### Data Flow

**Query Execution:**
1. API receives request → `app/api/v1/queries.py`
2. Calls `execute_query_with_service()` → wraps `database_service.execute_query()`
3. Service validates SQL → `sql_validator.py`
4. Service gets adapter from registry → `adapters/registry.py`
5. Adapter executes query using database-specific driver
6. Result returned through service layer → API response

**Metadata Extraction:**
1. API request → `app/api/v1/databases.py`
2. Calls `fetch_metadata()` → `app/services/metadata.py`
3. Checks cache (24hr TTL) in SQLite database
4. If stale/missing: calls `database_service.extract_metadata()`
5. Adapter queries information_schema
6. Results cached and returned

## Code Conventions

### Backend (Python/FastAPI)
- **Type hints**: Required for all function signatures
- **Async/await**: All database operations are async
- **Error handling**: Use `SqlValidationError` for SQL issues, `HTTPException` for API errors
- **Logging**: Use `logging` module, not `print()`
- **API routes**: Use `sqlmodel` for request/response models in `app/models/schemas.py`
- **Database models**: Use SQLModel classes in `app/models/database.py`, `app/models/query.py`

### Frontend (React/TypeScript)
- **Components**: Functional components with hooks
- **State**: React hooks (useState, useEffect)
- **API calls**: Axios via `app/services/`
- **Types**: TypeScript interfaces in `app/types/`
- **Styling**: Tailwind CSS classes (camelCase for dynamic styles)
- **Design System**: MotherDuck-inspired (sunbeam yellow #FFDE00, 2px black borders)

### Naming Conventions
- **Python**: `snake_case` for functions/variables, `PascalCase` for classes
- **TypeScript/React**: `PascalCase` for components, `camelCase` for functions/variables
- **API endpoints**: `kebab-case` for URL parameters (e.g., `/api/v1/dbs/{name}`)
- **Database**: `snake_case` for table/column names

## Important File Locations

### Backend Structure
```
backend/
├── app/
│   ├── api/v1/              # API route definitions
│   │   ├── databases.py     # Database connection endpoints
│   │   ├── queries.py       # Query execution endpoints
│   │   └── export.py        # Data export endpoint (CSV/JSON)
│   ├── adapters/            # Database abstraction layer
│   │   ├── base.py         # DatabaseAdapter ABC + data types
│   │   ├── registry.py     # Adapter factory and pool management
│   │   ├── postgresql.py   # PostgreSQL adapter
│   │   └── mysql.py        # MySQL adapter
│   ├── services/           # Business logic
│   │   ├── database_service.py  # Main facade
│   │   ├── sql_validator.py    # SQL validation/transformation
│   │   ├── export_service.py   # Query result export (CSV/JSON)
│   │   ├── nl2sql.py           # Natural language to SQL
│   │   └── metadata.py         # Metadata caching
│   ├── models/             # Data models
│   │   ├── database.py     # DatabaseConnection, DatabaseType
│   │   ├── query.py        # QueryHistory, QuerySource
│   │   └── schemas.py      # Pydantic models for API
│   └── main.py             # FastAPI app initialization
├── tests/                  # Backend tests
└── alembic/               # Database migrations
```

### Frontend Structure
```
frontend/
└── src/
    ├── components/         # Reusable components
    │   ├── SqlEditor.tsx         # Monaco SQL editor
    │   ├── NaturalLanguageInput.tsx  # NL input component
    │   ├── ResultTable.tsx        # Query results display
    │   ├── DatabaseSidebar.tsx   # Database list sidebar
    │   └── MetadataTree.tsx      # Database metadata tree
    ├── pages/              # Page components
    │   ├── Home.tsx            # Main query interface
    │   └── databases/          # Database-specific pages
    ├── services/           # API integration
    │   └── api.ts             # Axios client setup
    └── types/              # TypeScript types
```

## Adding Database Support

To add a new database type (e.g., Oracle):

1. **Create adapter**: `app/adapters/oracle.py` implementing `DatabaseAdapter`
2. **Update enum**: Add `ORACLE = "oracle"` to `DatabaseType` in `app/models/database.py`
3. **Register**: Add `self.register(DatabaseType.ORACLE, OracleAdapter)` in `adapters/registry.py`
4. **Update parser**: Add URL detection in `app/utils/db_parser.py` (if needed)
5. **Test**: Create `tests/adapters/test_oracle.py`

Reference: `docs/QUICK_REFERENCE.md` has detailed examples.

## Environment Setup

### Required
- Python 3.12+
- Node.js (LTS)
- `uv` Python package manager
- OPENAI_API_KEY in `backend/.env`

### Configuration Files
- `backend/.env` - Backend environment variables
- `backend/pyproject.toml` - Python dependencies
- `frontend/package.json` - Node dependencies

## Database Migration Process

1. Create migration: `make db-migrate MESSAGE="description"`
2. Review generated file in `backend/alembic/versions/`
3. Apply migration: `make db-upgrade`
4. Rollback if needed: `make db-downgrade REVISION=previous`

## Testing Strategy

### Backend Tests
- **Unit tests**: Test individual functions/services
- **Integration tests**: Test API endpoints with test database
- **Adapter tests**: Each adapter has dedicated tests
- Run with: `make test-backend`

### Frontend Tests
- **Component tests**: Test React components
- **Integration tests**: Test API integration
- Run with: `make test-frontend`

### API Testing
Use REST Client extension with `fixtures/test.rest` file for manual API testing.

## Key Dependencies

### Backend
- `fastapi` - Web framework
- `sqlmodel` - ORM and validation
- `asyncpg` - PostgreSQL async driver
- `aiomysql` - MySQL async driver
- `openai` - Natural language to SQL
- `sqlglot` - SQL parsing/validation
- `alembic` - Database migrations

### Frontend
- `react` + `typescript` - Core framework
- `@refinedev/core` - Admin framework
- `antd` - UI components
- `@monaco-editor/react` - SQL editor
- `axios` - HTTP client

## Common Issues

1. **Port already in use**: Backend uses 8000, frontend uses 5173
2. **OPENAI_API_KEY missing**: Natural language queries won't work without valid key
3. **Database connection fails**: Check URL format and database accessibility
4. **Metadata cache stale**: Call refresh endpoint or set `refresh=true` parameter
5. **Adapter not found**: Ensure database type is registered in `adapters/registry.py`

## Documentation

- `docs/ARCHITECTURE_REDESIGN.md` - Detailed architecture explanation
- `docs/QUICK_REFERENCE.md` - Adapter development quick reference
- `docs/IMPLEMENTATION_GUIDE.md` - Implementation details
- `backend/app/adapters/README.md` - Adapter development guide

---
> Source: [Yangyoung-max/Ai-Demo-Tool](https://github.com/Yangyoung-max/Ai-Demo-Tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
