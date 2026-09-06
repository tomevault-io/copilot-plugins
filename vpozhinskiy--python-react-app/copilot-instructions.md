## python-react-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Full-stack web application with Django backend, React frontend, PostgreSQL database, and Nginx reverse proxy. All services run in Docker containers with hot-reload enabled for development.

**Tech Stack:**
- Backend: Django 5.2 + Django REST Framework
- Frontend: React with TypeScript
- Database: PostgreSQL 16
- Web Server: Nginx (reverse proxy)
- Build: CRACO (Create React App Configuration Override)

## Common Commands

### Essential Docker Commands
```bash
# Build all images
make build

# Start all services (foreground)
make up

# Stop all services
make down

# View logs from all services
make logs

# View specific service logs
make logs-backend
make logs-nginx
make logs-postgres

# Check service status
make ps
make health

# Clean everything (removes containers and volumes)
make clean

# Rebuild from scratch (no cache)
make rebuild
```

### Backend Development
```bash
# Run migrations (after model changes)
make makemigrations
make migrate

# Create superuser for admin panel
make createsuperuser

# Run tests
make test

# Access Django shell
make shell

# Access backend container bash
make exec-backend

# Access PostgreSQL CLI
make exec-postgres
```

### Frontend Development
```bash
# Frontend runs automatically with hot-reload on port 3000
# To restart frontend after changes:
docker-compose restart frontend

# Frontend npm scripts (run inside container):
# npm start   - Start dev server
# npm build   - Build production bundle
# npm test    - Run tests
```

### Database Operations
```bash
# Direct PostgreSQL access
docker-compose exec postgres psql -U app_user -d app_db

# Or use the make command
make exec-postgres
```

## Architecture

### Service Architecture
```
Client → Nginx (Port 80) → Backend (Port 8000)
                         → Frontend (Port 3000 - dev only)

Backend ↔ PostgreSQL (Port 5432, isolated network)
```

### Network Isolation
- **backend_network**: PostgreSQL ↔ Backend (database isolated from public)
- **frontend_network**: Nginx ↔ Backend (public-facing)

### Django Backend Structure (Clean Architecture)

The backend follows a layered architecture pattern:

```
backend/apps/submissions/
├── models/              # Domain models (pure data structures)
│   └── submission.py    # Submission model with DB indexes
├── dto/                 # Data Transfer Objects
│   └── dtos.py          # SubmissionDTO for service layer
├── serializers/         # DRF serializers (request/response validation)
│   └── serializers.py   # SubmissionRequestSerializer, HistoryResponseSerializer
├── repositories/        # Data access layer
│   └── submission_repository.py  # Database operations only
├── services/            # Business logic layer
│   ├── submission_service.py     # Validation, processing delay, orchestration
│   └── history_service.py        # History retrieval logic
├── views/               # Presentation layer (HTTP handling)
│   ├── submission_view.py        # POST /api/submit
│   └── history_view.py           # GET /api/history
└── urls.py              # URL routing
```

**Layering Principles:**
- **Models**: Pure Django models, no business logic
- **DTOs**: Simple data structures for passing data between layers
- **Repositories**: Only database operations (CRUD), no business logic
- **Services**: All business logic (validation, delays, orchestration)
- **Views**: HTTP handling only (request/response), dependency injection
- **Serializers**: Input validation and response serialization

**Dependency Flow**: Views → Services → Repositories → Models

When adding new features:
1. Define model in `models/`
2. Create DTO in `dto/`
3. Create repository methods in `repositories/`
4. Implement business logic in `services/`
5. Create serializers for request/response in `serializers/`
6. Create view in `views/` that wires everything together
7. Add route in `urls.py`

### React Frontend Structure

```
frontend/src/
├── api/                 # API client configuration
│   └── client.ts        # Axios instance with interceptors
├── components/          # Reusable UI components
│   ├── common/          # Generic components (Button, Link, Spinner)
│   ├── form/            # Form components (Input, Select, FieldError)
│   └── table/           # Table components
├── pages/               # Page-level components
│   ├── HomePage/        # Landing page with navigation
│   ├── SubmissionPage/  # Form submission page
│   └── HistoryPage/     # View submission history
├── features/            # Feature-specific logic
├── hooks/               # Custom React hooks
│   └── useApi.ts        # API call hook with loading/error states
├── types/               # TypeScript type definitions
├── utils/               # Utility functions
└── styles/              # Global styles
```

**Component Structure**: Each component has its own directory with `ComponentName.tsx` and `index.ts`

### API Endpoints

**Backend (Django REST Framework):**
- `POST /api/submit` - Submit form data
  - Request: `{date: string, first_name: string, last_name: string}`
  - Validation: No whitespace allowed in names
  - Processing: Random delay 0-3 seconds
  - Response: `{success: boolean, error?: object}`

- `GET /api/history` - Get submission history
  - Response: Array of submissions ordered by date (desc), first_name, last_name

- `GET /health/` - Health check endpoint

**Frontend Routes:**
- `/` - Home page
- `/submission` - Submission form
- `/history` - Submission history table

### Nginx Configuration

- Rate limiting: 10 req/s for API, 2 req/s for `/api/submit`
- Static files: React build at `/`, Django static at `/django-static/`
- Security headers: X-Frame-Options, X-Content-Type-Options, X-XSS-Protection
- Gzip compression enabled
- Health check: `/nginx-health`

### Development Workflow

**Hot Reload:**
- Backend: Django runserver with volume mount (`./backend:/app`)
- Frontend: React dev server with polling enabled (WATCHPACK_POLLING, CHOKIDAR_USEPOLLING)

**Making Changes:**

1. **Backend changes** (models, views, services):
   - Edit files in `backend/`
   - Django auto-reloads
   - If models changed: `make makemigrations && make migrate`

2. **Frontend changes** (components, pages):
   - Edit files in `frontend/src/`
   - React auto-reloads in browser
   - TypeScript files compile automatically

3. **Database schema changes**:
   ```bash
   # After modifying models
   make makemigrations
   make migrate
   ```

4. **Adding new Python dependencies**:
   - Add to `backend/requirements.txt`
   - Rebuild: `docker-compose build backend`

5. **Adding new npm packages**:
   - Exec into frontend container: `docker-compose exec frontend /bin/sh`
   - Run: `npm install package-name`
   - Or add to `package.json` and rebuild

### Environment Variables

Required in `.env` (copy from `.env.example`):
- `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD` - Database credentials
- `DB_HOST=postgres`, `DB_PORT=5432` - Database connection
- `DJANGO_SECRET_KEY` - Django secret (generate new for production)
- `DJANGO_DEBUG` - `True` for dev, `False` for production
- `DJANGO_ALLOWED_HOSTS` - Comma-separated allowed hosts

### Testing

**Backend Tests (pytest + pytest-django):**

The backend has comprehensive test coverage (96.81%) with 49 tests organized in three categories:

```bash
# Run all tests (49 tests)
make test

# Run specific test types
make test-unit         # Unit tests (18 tests) - services, serializers
make test-integration  # Integration tests (18 tests) - repository, models
make test-api          # API tests (13 tests) - endpoints end-to-end

# Run with coverage report
make test-coverage     # Generates HTML report in backend/htmlcov/

# Run tests in parallel (faster)
make test-fast
```

**Test Structure:**
```
backend/tests/
├── conftest.py              # Global fixtures
├── factories.py             # Factory-boy factories for test data
├── unit/                    # Unit tests (mocked dependencies)
│   ├── test_submission_service.py
│   ├── test_history_service.py
│   └── test_serializers.py
├── integration/             # Integration tests (real DB)
│   ├── test_submission_repository.py
│   └── test_models.py
└── api/                     # End-to-end API tests
    ├── test_submission_endpoint.py
    └── test_history_endpoint.py
```

**Test Dependencies:**
- pytest, pytest-django, pytest-cov
- factory-boy + Faker (test data generation)
- pytest-xdist (parallel execution)
- freezegun (time mocking)

All test dependencies are in `backend/requirements-dev.txt`.

**Frontend:**
```bash
docker-compose exec frontend npm test
```

### Health Checks

All services have health checks configured:
- PostgreSQL: `pg_isready` every 10s
- Backend: HTTP check on `/health/` every 30s (40s start period)
- Nginx: HTTP check on `/nginx-health` every 30s

Services depend on upstream health (backend waits for postgres, nginx waits for backend).

### Troubleshooting

**Database connection errors:**
```bash
# Check postgres health
docker-compose ps postgres

# Test connection from backend
docker-compose exec backend nc -zv postgres 5432
```

**Backend not responding:**
```bash
make logs-backend
curl http://localhost/api/
```

**Frontend not updating:**
```bash
# Check polling is enabled
docker-compose logs frontend | grep -i polling
docker-compose restart frontend
```

**Nginx 502 errors:**
```bash
# Backend might not be healthy
make health
make logs-nginx
```

### URLs During Development

- Frontend (direct): http://localhost:3000 (React dev server)
- Frontend (via Nginx): http://localhost
- Backend API: http://localhost/api/
- Django Admin: http://localhost/admin/
- PostgreSQL: localhost:5432 (from host), postgres:5432 (from containers)

### Important Notes

- Frontend `node_modules` is mounted as anonymous volume to prevent overwriting
- Static files collected to shared volume in entrypoint.sh
- Backend entrypoint runs migrations and collectstatic automatically
- All containers use non-root users for security
- Database data persists in named volume `postgres_data`

---
> Source: [vpozhinskiy/python_react_app](https://github.com/vpozhinskiy/python_react_app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
