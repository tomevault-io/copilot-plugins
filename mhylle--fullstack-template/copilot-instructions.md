## fullstack-template

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a full-stack notes application with:
- **Backend**: NestJS (Node.js/TypeScript) REST API on port 3000
- **Frontend**: Angular 19 (TypeScript/SCSS) standalone components

## Project Structure

```
mynotes/
├── backend/          # NestJS backend application
│   ├── src/         # Source code (controllers, services, modules)
│   └── test/        # E2E and unit tests
├── frontend/        # Angular frontend application
│   └── src/         # Source code (components, services, routes)
└── package.json     # Root package (contains Angular CLI)
```

## Development Commands

### Backend (NestJS)

Navigate to `backend/` directory for all backend operations:

```bash
cd backend

# Install dependencies
npm install

# Development
npm run start:dev     # Watch mode with auto-reload
npm run start:debug   # Debug mode with watch

# Build
npm run build         # Compile TypeScript to dist/

# Production
npm run start:prod    # Run compiled code from dist/

# Testing
npm run test          # Unit tests (Jest)
npm run test:watch    # Unit tests in watch mode
npm run test:cov      # Unit tests with coverage
npm run test:e2e      # End-to-end tests

# Code Quality
npm run lint          # ESLint with auto-fix
npm run format        # Prettier formatting
```

### Frontend (Angular)

Navigate to `frontend/` directory for all frontend operations:

```bash
cd frontend

# Install dependencies
npm install

# Development
npm start             # Start dev server (default: http://localhost:4200)

# Build
npm run build         # Production build to dist/frontend
npm run watch         # Development build with watch mode

# Testing
npm test              # Karma unit tests
```

## Architecture Notes

### Backend (NestJS)

- **Module-based architecture**: Features organized in modules with controllers, services, and providers
- **Dependency injection**: Use constructor injection for services
- **Port configuration**: Backend runs on port 3000 (configurable via PORT env var)
- **Entry point**: `src/main.ts` bootstraps the NestJS application
- **Root module**: `src/app.module.ts`

### Frontend (Angular)

- **Standalone components**: Angular 19 uses standalone component architecture (no NgModules)
- **SCSS styling**: Component styles use SCSS syntax
- **Routing**: Configured in `src/app/app.routes.ts`
- **Configuration**: `src/app/app.config.ts` provides application-wide providers
- **Component prefix**: `app` (defined in `angular.json`)

## Environment Configuration

The application uses Angular environment files for configuration across different deployment scenarios:

### Environment Files

Located in `frontend/src/environments/`:

- **`environment.ts`** - Default (local development)
  - Backend API: `http://localhost:3000`
  - Used when running frontend locally with `npm start`

- **`environment.docker.ts`** - Docker development
  - Backend API: `http://localhost:11002`
  - Used when running in Docker Compose
  - Configured via `angular.json` with `--configuration=docker`

- **`environment.production.ts`** - Production
  - Backend API: `/api` (relative URL)
  - Used for production builds
  - Assumes nginx reverse proxy handles routing

### Using Environments

**Local Development:**
```bash
cd frontend
npm start  # Uses environment.ts (backend on localhost:3000)
```

**Docker Development:**
```bash
docker compose up -d  # Uses environment.docker.ts (backend on localhost:11002)
```

**Production Build:**
```bash
cd frontend
npm run build  # Uses environment.production.ts (relative /api URL)
```

### Adding New Configuration

To add configuration values:
1. Add property to all environment files
2. Import in services: `import { environment } from '../environments/environment'`
3. Use in code: `environment.yourProperty`

## Authentication Configuration

The application supports **dual-mode authentication** for flexible development:

### Authentication Modes

1. **Local Mode** (Default for Development)
   - Uses centralized auth service at `https://mhylle.com/api/auth`
   - Allows development without local database user management
   - Shares users across all apps in mhylle.com infrastructure
   - Set `AUTH_MODE=local` in environment

2. **Production Mode**
   - Uses local PostgreSQL database for user management
   - Complete auth independence
   - Set `AUTH_MODE=production` in environment

### Configuration

**Backend Environment Variables** (`.env`):
```env
AUTH_MODE=local                              # 'local' or 'production'
AUTH_SERVICE_URL=https://mhylle.com/api/auth # External auth service URL
JWT_SECRET=your-secret-key-change-in-production
```

### Implementation Pattern

The backend uses a **factory provider pattern** to switch between auth services:
- `LocalAuthService` - Proxies requests to external auth service
- `ProductionAuthService` - Handles auth locally with database
- Both implement `IAuthService` interface for consistency

### Guest Usage

**By default, the application allows guest usage** - no login required:
- All features accessible without authentication
- Optional login for personalization
- User data tied to authenticated sessions when logged in

## Testing Strategy

- **Backend**: Jest for unit tests, separate config for E2E tests (`test/jest-e2e.json`)
- **Frontend**: Karma + Jasmine for unit tests
- Test files follow `*.spec.ts` naming convention for both projects

## Key Dependencies

- **Backend**: NestJS 11, TypeScript 5.7, RxJS 7.8
- **Frontend**: Angular 19, TypeScript 5.7, RxJS 7.8
- **Database**: PostgreSQL 17 with pgvector extension
- **Build tools**: SWC (backend), esbuild via Angular CLI (frontend)

## Docker Setup

### Port Configuration

- **Frontend**: Port 11001 (http://localhost:11001)
- **Backend**: Port 11002 (http://localhost:11002)
- **PostgreSQL**: Port 11003 (localhost:11003)

### Docker Compose Services

The project includes three Docker services:
1. **postgres**: PostgreSQL 17 with pgvector extension (required)
2. **backend**: NestJS API server (optional for development)
3. **frontend**: Angular dev server (optional for development)

### Common Docker Commands

```bash
# Start all services (database + backend + frontend)
docker compose up -d

# Start only the database (for local development)
docker compose up -d postgres

# View logs
docker compose logs -f [service_name]

# Stop all services
docker compose down

# Stop all services and remove volumes (deletes database data)
docker compose down -v

# Rebuild services after Dockerfile changes
docker compose up -d --build

# Check service status
docker compose ps
```

### Development Workflows

#### Option 1: Full Docker Setup

Run everything in Docker:

```bash
docker compose up -d
```

Access:
- Frontend: http://localhost:11001
- Backend: http://localhost:11002
- Database: localhost:11003

#### Option 2: Hybrid Development (Recommended)

Run only the database in Docker, develop backend and frontend locally:

```bash
# Start database only
docker compose up -d postgres

# In separate terminals:
cd backend && npm run start:dev      # Backend on port 3000
cd frontend && npm start              # Frontend on port 4200
```

For local development, backend connects to database at `localhost:11003`.
Copy `backend/.env.example` to `backend/.env` and adjust settings if needed.

#### Option 3: Database + One Service

Run database and backend in Docker, frontend locally:

```bash
docker compose up -d postgres backend
cd frontend && npm start
```

### Database Configuration

**PostgreSQL with pgvector**:
- Image: `pgvector/pgvector:pg17`
- Database: `mynotes`
- User: `mynotes_user`
- Password: `mynotes_password`
- Port: 11003
- Extension: pgvector (auto-enabled via `docker/init-db.sql`)

**Connecting to the database**:
```bash
# From host machine
psql -h localhost -p 11003 -U mynotes_user -d mynotes

# From within Docker network
psql -h postgres -p 5432 -U mynotes_user -d mynotes
```

### Environment Variables

**Backend** (`backend/.env` for local development):
- `NODE_ENV`: development
- `PORT`: 3000
- `DATABASE_HOST`: localhost (or postgres in Docker)
- `DATABASE_PORT`: 11003 (or 5432 in Docker)
- `DATABASE_NAME`: mynotes
- `DATABASE_USER`: mynotes_user
- `DATABASE_PASSWORD`: mynotes_password

### CORS Configuration

Backend is configured to accept requests from:
- http://localhost:11001 (Docker frontend)
- http://localhost:4200 (Local frontend)

### Notes

- Database volume `postgres_data` persists data between container restarts
- Backend and frontend use volume mounts in development for live code reloading
- The `pgvector` extension is automatically enabled on first database initialization
- Use `docker compose down -v` to completely reset the database

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mhylle)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mhylle)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
