## ogon2025

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OGON 2025 is a modern monolith application featuring:
- **Backend**: Fastify API with TypeScript, LangChain, and LangGraph for AI capabilities
- **Frontend**: React SPA with Vite, Tailwind CSS, and TanStack Query
- **Architecture**: Turborepo monorepo structure for efficient builds
- **Database**: PostgreSQL for data persistence
- **Vector Database**: Supabase with pgvector for embeddings and similarity search

## Development Commands

### Root Level Commands
```bash
yarn install    # Install all dependencies for the monorepo
yarn dev        # Start all apps in development mode
yarn build      # Build all apps
yarn lint       # Run ESLint across all apps
yarn test       # Run tests across all apps
yarn typecheck  # Run TypeScript type checking
yarn clean      # Clean build artifacts
```

### Quick Start (Docker - Recommended)
```bash
# Clone and setup
git clone <repository-url>
cd ogon2025

# Configure environment
cp .env.example apps/backend/.env.local
# Edit apps/backend/.env.local with your OpenAI API key

# Start all services
docker compose -f docker-compose.local.yml up -d

# Access application
# Frontend: http://localhost:8080
# Backend: http://localhost:3000
# API Docs: http://localhost:3000/documentation
```

### Docker Development Commands
```bash
# Start all services with hot reload
docker compose -f docker-compose.local.yml up -d

# View logs
docker compose -f docker-compose.local.yml logs -f

# Restart specific service
docker compose -f docker-compose.local.yml restart backend

# Stop all services
docker compose -f docker-compose.local.yml down

# View backend logs
docker logs ogon2025-backend-1 --tail 50 -f
```

### Manual Development (Alternative)
```bash
# Backend only
cd apps/backend && yarn dev  # Port 3000

# Frontend only  
cd apps/frontend && yarn dev  # Port 5173

# All services
yarn dev  # Starts both backend and frontend
```

## Architecture & Structure

### Monorepo Layout
```
ogon2025/
├── apps/
│   ├── backend/      # Fastify API server
│   └── frontend/     # React SPA
├── packages/         # Shared packages (future)
├── turbo.json        # Turborepo configuration
└── package.json      # Root workspace configuration
```

### Backend Architecture
- **Framework**: Fastify with TypeScript
- **AI Integration**: LangChain and LangGraph for agent workflows
- **Database**: PostgreSQL with @fastify/postgres and Prisma ORM
- **WebSocket**: Real-time communication with @fastify/websocket
- **Query Execution**: Secure Query Executor service with Prisma for safe SQL execution
- **Vector Database**: Supabase with pgvector for embeddings storage and similarity search
- **API Documentation**: Auto-generated Swagger UI at /documentation
- **Key Directories**:
  - `src/agents/`: LangGraph agent implementations
  - `src/services/`: Business logic (database, websocket, RAG, Supabase, query-executor)
  - `src/routes/`: API route handlers
  - `src/db/`: Database schemas and migrations
  - `src/types/`: TypeScript type definitions
  - `src/utils/`: Utility functions including vector operations

### Frontend Architecture
- **Framework**: React 19 with TypeScript
- **Build Tool**: Vite 6 for fast development
- **Styling**: Tailwind CSS v4 with PostCSS (@tailwindcss/postcss)
- **State Management**: Zustand
- **Data Fetching**: TanStack Query
- **Routing**: React Router v7
- **API Configuration**: Environment-based API URL configuration with `VITE_API_URL`

## Key Technical Decisions

1. **ES Modules**: Both backend and frontend use ES modules (`"type": "module"`)
2. **TypeScript Strict Mode**: Enabled for type safety (TypeScript 5.8)
3. **Monorepo with Turborepo**: Efficient caching and parallel builds (Turborepo 2.5.4)
4. **API Proxy**: Frontend dev server proxies /api to backend (localhost:3000)
5. **React 19**: Latest React with improved server components and concurrent features
6. **TailwindCSS v4**: New architecture with @tailwindcss/postcss plugin

## Recent Updates (January 2025)

**Major Version Updates:**
- **React 18 → 19**: Updated with new concurrent features and server components
- **TailwindCSS 3.4 → 4.1**: Migrated to new PostCSS plugin architecture (@tailwindcss/postcss)
- **Vite 6.0 → 6.3**: Latest build optimizations and improvements
- **TypeScript 5.4 → 5.8**: Enhanced type checking and performance
- **Turborepo 2.3 → 2.5**: Improved caching and build performance
- **LangChain packages**: Updated to latest 0.3.x versions

**Important Configuration Changes:**
- **PostCSS Config**: Updated to use `@tailwindcss/postcss` instead of `tailwindcss` directly
- **React Types**: Updated to React 19 compatible types (@types/react@^19.0.0)
- **Node Types**: Updated to latest LTS version compatibility (@types/node@^22.15.29)

**Breaking Changes Handled:**
- TailwindCSS v4 PostCSS plugin migration completed
- React 19 type compatibility ensured
- All ESLint and TypeScript checks passing

**GitHub Actions Security Fixes:**
- Added Corepack enablement for proper Yarn 4.1.1 usage in CI/CD
- Made Slack notifications conditional to prevent security warnings
- Added explicit permissions for paths-filter and other actions
- Ensured package manager version consistency across all workflows

## Development Workflow

### Docker Development (Recommended)
1. **Environment Setup**: Copy `.env.example` to `apps/backend/.env.local`
2. **Configure OpenAI**: Edit `.env.local` with your OpenAI API key
3. **Start Services**: `docker compose -f docker-compose.local.yml up -d`
4. **Access Apps**: Frontend at http://localhost:8080, Backend at http://localhost:3000

### Manual Development (Alternative)
1. **Environment Setup**: Copy `.env.example` to `apps/backend/.env.local`
2. **Database Setup**: Start PostgreSQL locally and create database
3. **Dependencies**: Run `yarn install` at root to install all dependencies
4. **Development**: Run `yarn dev` to start both frontend and backend

## Code Style & Quality

### EditorConfig Settings
- **Indentation**: 2 spaces for JS/JSON/YML files
- **Line Endings**: LF (Unix-style)
- **Final Newline**: Always insert
- **Charset**: UTF-8

### ESLint Configuration
- **Backend**: Node.js environment with TypeScript rules
  - Globals: Node.js globals enabled
  - Unused vars: Must prefix with underscore (_)
  - Located in: `apps/backend/eslint.config.js`

- **Frontend**: Browser environment with React rules
  - Globals: Browser globals enabled
  - React: JSX runtime automatic (no import needed)
  - React Refresh: Enabled for HMR
  - Located in: `apps/frontend/eslint.config.js`

### TypeScript Configuration
- **Strict Mode**: Enabled for type safety
- **Module**: ES2022 for backend, ESNext for frontend
- **Path Aliases**: `@/*` maps to `src/*` in both apps

## Important Patterns

### Backend Patterns
- Fastify plugins for route organization
- Zod for runtime validation
- LangGraph StateGraph for agent workflows
- PostgreSQL for session/message persistence
- Supabase vector service for embeddings and similarity search
- Vector utilities for mathematical operations (cosine similarity, normalization)
- Schema enrichment with LLM for improved search accuracy
- Query expansion for better semantic understanding

### Frontend Patterns
- Component-based architecture
- Custom hooks for business logic
- Service layer for API calls
- Type-safe API integration with TypeScript

## Local Development Setup

### Environment Configuration

The project uses a separated environment configuration approach for better developer experience:

1. **Production Requirements**: All environment variables are required in production
2. **Development Flexibility**: Optional services (like Supabase) are gracefully handled in development
3. **Local Configuration**: Uses `.env.local` for minimal local development setup

### Quick Start for Local Development

```bash
# 1. Start services with automatic setup and logging
./start-local.sh

# 2. Initialize database (first time only)
cd apps/backend && ./scripts/local-dev.sh init

# 3. Access the application
# Frontend: http://localhost:8080 (nginx with API proxy)
# Backend: http://localhost:3000
# API Docs: http://localhost:3000/documentation
```

### Development Logging & Debugging

The development environment provides comprehensive logging for debugging API requests:

**Frontend Logging:**
- Axios interceptors log all HTTP requests/responses in browser console
- Nginx access logs for API requests in frontend container
- Component-level logging for chat interactions

**Backend Logging:**
- Structured JSON logs with request tracing
- Detailed logging for `/api/agent/chat` endpoint
- CORS and authentication details
- Request/response timing and metadata

**Request Flow:**
1. Frontend (React) → Axios → Nginx proxy → Backend
2. All stages are logged for easy debugging
3. Use browser dev tools to see frontend logs
4. Use `docker logs <container>` to see backend logs

**Debugging Tips:**
```bash
# Watch backend logs
docker logs -f ogon2025-backend-1

# Watch frontend logs  
docker logs -f ogon2025-frontend-1

# Access containers
docker exec -it ogon2025-backend-1 sh
docker exec -it ogon2025-frontend-1 sh
```

### Environment Files

- `.env.local` - Minimal configuration for local development
- `.env.development` - Development environment with all services
- `.env.production` - Production environment (never commit)
- `.env.example` - Template with all available options

### Working with Optional Services

In development, the following services are optional:
- **Supabase Vector Database**: Set `SUPABASE_URL` and `SUPABASE_KEY` to enable
- **OpenAI**: Set `OPENAI_API_KEY` to enable AI features
- **Redis**: Automatically available via Docker Compose

### Database Management

```bash
cd apps/backend

# Initialize local environment
./scripts/local-dev.sh init

# Run Prisma commands with correct DATABASE_URL
./scripts/local-dev.sh generate  # Generate Prisma Client
./scripts/local-dev.sh push      # Push schema to database
./scripts/local-dev.sh migrate   # Run migrations
./scripts/local-dev.sh studio    # Open Prisma Studio

# Reset database (WARNING: deletes all data)
./scripts/local-dev.sh migrate:reset
```

### Docker Development

The local development setup uses optimized Docker configuration:
- Named volumes for `node_modules` to prevent dependency issues
- Hot reload enabled for both frontend and backend
- Simplified startup without automatic Prisma commands
- Health checks that work without all services

**Docker Files**:
- `Dockerfile` - Production build (multi-stage, optimized)
- `Dockerfile.dev` - Development build (includes dev dependencies, hot reload)
- Local development uses `Dockerfile.dev` for better DX
- Production deployments use `Dockerfile` for smaller images

### Troubleshooting Local Development

1. **Supabase errors on startup**
   - This is expected in local development
   - Vector search features will be disabled
   - Add credentials to enable: edit `.env.local`

2. **Database connection from host**
   - Use `localhost:5432` when running commands from host
   - Use `postgres:5432` when running inside containers
   - The `local-dev.sh` script handles this automatically

3. **Hot reload not working**
   - Check volume mounts in `docker-compose.local.yml`
   - Ensure source files are properly mounted
   - Restart containers if needed

4. **Prisma Client not found**
   - Run: `cd apps/backend && ./scripts/local-dev.sh generate`
   - This generates all required Prisma clients

## Common Tasks

### Adding a New API Endpoint
1. Create route handler in `apps/backend/src/routes/`
2. Register route in `apps/backend/src/index.ts` with prefix
3. Add Swagger schema for auto-documentation
4. Add TypeScript types in frontend `apps/frontend/src/types/`
5. Create service method in `apps/frontend/src/services/`

### Working with LangGraph Agents
1. Define state schema using Annotation
2. Create nodes as async functions
3. Build graph with StateGraph
4. Compile with checkpointer for memory
5. Handle errors gracefully in route handlers

### Database Operations
- Schema files: `apps/backend/src/db/`
- Migrations: `psql -d ogon2025 -f <migration.sql>`
- Connection pool via @fastify/postgres
- Always release connections after use

### Working with Query Executor Service
1. **Import the service**: `import { queryExecutor } from '@/services/query-executor'`
2. **Execute queries safely**: `await queryExecutor.execute(sql, { format: 'table', maxRows: 100 })`
3. **Validate SQL**: `const processedSQL = queryExecutor.validateAndProcessSQL(sql)`
4. **Format results**: `queryExecutor.formatResults(data, fields, 'chart')`
5. **Configure timeouts**: Set `timeout` option (default: 30s)
6. **Security features**:
   - Only SELECT and WITH queries allowed
   - Automatic LIMIT injection if missing
   - SQL injection prevention via Prisma
   - Query timeout protection
   - Multi-schema support (main, wallet, pam, games, kyc)

### Working with Prisma Multi-Database Setup
1. **Schema Locations**: 
   - Main: `apps/backend/prisma/schema.prisma` (DATABASE_URL)
   - Wallet: `apps/backend/prisma/wallet/schema.prisma` (WALLET_DATABASE_URL) 
   - PAM: `apps/backend/prisma/pam/schema.prisma` (PAM_DATABASE_URL)
   - Games: `apps/backend/prisma/games/schema.prisma` (GAMES_DATABASE_URL)
   - KYC: `apps/backend/prisma/kyc/schema.prisma` (KYC_DATABASE_URL)
2. **Client Generation**: `yarn prisma:generate` (generates all 5 clients)
3. **Database Push**: `yarn prisma:push` (pushes all schemas to respective databases)
4. **Migrations**: `yarn prisma:migrate` (runs migrations for all databases)
5. **Prisma Studio**: `yarn prisma:studio` (opens 5 instances on ports 5555-5559)
6. **Client Usage**: Import from generated client directories in `apps/backend/prisma/generated/`
7. **Important**: Generated clients are excluded from version control via `.gitignore`

### Working with Supabase Vector Database
1. Import the service: `import { supabaseVectorService } from '@/services/supabase/index.js'`
2. Initialize once: `await supabaseVectorService.initialize()`
3. Store embeddings: `await supabaseVectorService.insertSchemaEmbedding({...})`
4. Search similar schemas: `await supabaseVectorService.searchSimilarSchemas(embedding, options)`
5. Use vector utilities for calculations: `import { cosineSimilarity, normalizeVector } from '@/utils/vector.js'`
6. See complete setup guide: `docs/infra/supabase-vector-setup.md`

### Schema Enrichment with LLM
1. **Automatic Enrichment**: Schemas are automatically enriched during embedding generation
2. **Business Metadata**: Display names, domains, descriptions, aliases, typical queries
3. **Usage Context**: Common filters, joins, aggregations, access patterns
4. **Data Quality**: Update frequency, sensitivity levels, deprecation status
5. **Enhanced Search**: Query expansion with synonyms for better results
6. **Script Usage**: `yarn tsx scripts/init-rag-embeddings.ts` (includes enrichment by default)
7. **Skip Enrichment**: Add `--skip-enrichment` flag for faster processing

### Vector Database Testing Scripts
```bash
# Test vector operations (comprehensive test suite)
cd apps/backend && yarn vector:test

# Set up Supabase vector schema (tables, indexes, functions) 
cd apps/backend && yarn vector:setup

# Interactive demo environment for testing and exploration
cd apps/backend && yarn vector:demo
```

**Important Notes:**
- All vector operations use Supabase exclusively (no local PostgreSQL vector storage)
- Scripts require SUPABASE_URL and SUPABASE_KEY environment variables
- Setup script provides SQL for manual execution in Supabase Dashboard
- Test script includes performance benchmarks and data validation

### Testing Patterns
1. **Unit Tests**: Test individual functions and utilities
2. **Integration Tests**: Test full service workflows (require real connections)
3. **CI Compatibility**: Integration tests automatically skip when `CI=true` to avoid credential requirements
4. **Local Development**: Use valid environment variables for full test suite
5. **Test Structure**: Use dynamic imports to prevent module loading errors in CI environments
6. **Query Executor Tests**: 
   - Mock Prisma client for unit tests: `yarn test src/services/query-executor`
   - 41 comprehensive test cases covering security, execution, formatting, and error handling
   - Uses Vitest with mocked database calls
   - Tests SQL validation, timeout handling, and result formatting

### Running Quality Checks
```bash
# Run all checks
yarn lint && yarn typecheck && yarn build

# Fix lint issues automatically (if possible)
cd apps/backend && npx eslint --fix src
cd apps/frontend && npx eslint --fix src
```

### Working with Database from Host (Local Development)
**Important**: When running Prisma commands from your host machine (not inside Docker), use the local development script:

```bash
# Setup database (generate client + push schema + seed)
cd apps/backend && ./scripts/local-dev.sh setup

# Individual commands
./scripts/local-dev.sh generate  # Generate Prisma Client
./scripts/local-dev.sh push     # Push schema to database
./scripts/local-dev.sh seed     # Seed database with data
./scripts/local-dev.sh studio   # Open Prisma Studio

# Direct commands with correct DATABASE_URL
DATABASE_URL=postgresql://ogon2025:changeme@localhost:5432/ogon2025 yarn prisma:generate
DATABASE_URL=postgresql://ogon2025:changeme@localhost:5432/ogon2025 yarn prisma:push
DATABASE_URL=postgresql://ogon2025:changeme@localhost:5432/ogon2025 yarn prisma:studio
```

**Note**: The `.env` file uses `postgres:5432` (Docker service name), but when running from host, you need `localhost:5432`.

## Environment Variables

### Backend Required Variables
- `DATABASE_URL`: PostgreSQL connection string
- `OPENAI_API_KEY`: For LangChain/OpenAI integration (required for AI features)
- `OPENAI_MODEL`: Model name (default: gpt-4-turbo-preview)
- `OPENAI_TEMPERATURE`: Model temperature (default: 0.7)
- `WEBSOCKET_PORT`: WebSocket server port (default: 3001)
- `WEBSOCKET_PATH`: WebSocket endpoint path (default: /ws)
- `PORT`: Server port (default: 3000)
- `NODE_ENV`: Environment (development/production/test)
- `LOG_LEVEL`: Fastify log level
- `CORS_ORIGIN`: Frontend URL for CORS
- `SUPABASE_URL`: Supabase project URL (optional - for vector database)
- `SUPABASE_KEY`: Supabase anon key (optional - for vector database)

### Frontend Variables
- `VITE_API_URL`: API base URL (e.g., 'https://api.nltosql.com' or '/api' for proxy)
  - **Development**: Uses proxy to backend via `/api` (default)
  - **Production**: Set to full API URL (e.g., 'https://api.nltosql.com')
  - **Build Time**: Injected via Docker build args and environment files
  - **Environment Files**:
    - `.env.development`: `VITE_API_URL=https://dev.api.nltosql.com`
    - `.env.production`: `VITE_API_URL=https://api.nltosql.com`
    - `.env.local`: For local development overrides (gitignored)

## Debugging Tips

### Docker Development Debugging
```bash
# View logs in real-time
docker logs ogon2025-backend-1 --tail 50 -f
docker logs ogon2025-frontend-1 --tail 50 -f

# Check container status
docker ps

# Inspect environment variables
docker exec ogon2025-backend-1 printenv | grep OPENAI

# Test API endpoints
curl -X POST http://localhost:3000/api/agent/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello!", "sessionId": "test-123"}'

# Test end-to-end through frontend proxy
curl -X POST http://localhost:8080/api/agent/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello!", "sessionId": "test-123"}'
```

### Common Issues
1. **OpenAI API Errors**
   - Verify `OPENAI_API_KEY` in `apps/backend/.env.local`
   - Check container environment: `docker exec ogon2025-backend-1 printenv | grep OPENAI`

2. **WebSocket Connection Failed**
   - Ensure WebSocket port 3001 is exposed and accessible
   - Check backend logs for WebSocket initialization messages

3. **Frontend Not Loading**
   - Verify nginx configuration in frontend container
   - Check CORS settings match frontend URL

4. **Database Connection Errors**
   - Ensure PostgreSQL container is running: `docker ps`
   - Verify DATABASE_URL uses correct host (`postgres` for Docker)

### Traditional Debugging
1. **Backend Debugging**
   - Fastify logger outputs to console
   - Check `LOG_LEVEL` in .env for verbosity
   - Use `fastify.log.error()` for error logging

2. **Frontend Debugging**
   - React DevTools for component inspection
   - Network tab for API call debugging
   - TanStack Query DevTools for query state

3. **TypeScript Issues**
   - Run `yarn typecheck` for detailed errors
   - Check `tsconfig.json` for path resolution
   - Ensure imports use correct extensions

## CI/CD and Deployment

### GitHub Actions Workflows

#### PR Validation (`.github/workflows/pr-validation.yml`)
- **Triggers**: Pull requests to main branch
- **Jobs**: lint, typecheck, build (Node 18/20/22), test, security scan
- **Services**: PostgreSQL for integration tests
- **Security**: Trivy vulnerability scanning, CodeQL analysis
- **Quality Gates**: All checks must pass before merge

#### Deployment (`.github/workflows/deploy.yml`)
- **Triggers**: Push to main, manual dispatch
- **Stages**: Validation → Build Images → Deploy Staging → Deploy Production
- **Environments**: Staging (auto), Production (manual approval)
- **Features**: Health checks, rollback on failure, notifications

#### Security (`.github/workflows/security.yml`)
- **Schedule**: Daily at 2 AM UTC
- **Scans**: Dependencies, code analysis, container images
- **Reports**: SARIF uploads to GitHub Security tab

### Deployment

**IMPORTANT**: Deployments are now handled via Dokploy (console.nltosql.com), not GitHub Actions. GitHub Actions workflows exist but are not actively used for deployment.

- **Production**: main branch → auto-deploy to https://nltosql.com
- **Development**: develop branch → auto-deploy to https://dev.nltosql.com
- **Local**: Use docker-compose with local override files

### Docker Configuration

#### Multi-stage Builds
- **Backend**: Node Alpine → Builder → Production (non-root)
- **Frontend**: Node Alpine → Builder → Nginx Alpine (non-root)
- **Security**: Vulnerability scanning, minimal attack surface
- **Health**: Health checks disabled for Dokploy compatibility (were causing restart loops)

#### Container Orchestration
- **Development**: Hot reload with volume mounts
- **Production**: Optimized images with proper networking
- **Services**: Backend, Frontend, PostgreSQL, Redis (optional)

#### Docker Compose Files
- `docker-compose.yml` - Base configuration used by Dokploy for all deployments
- `docker-compose.local.yml` - Local development with hot reload

### Environment Configuration

#### Local Development Environment Variables
```env
# apps/backend/.env.local
NODE_ENV=development
PORT=3000
LOG_LEVEL=debug
DATABASE_URL=postgresql://ogon2025:changeme@postgres:5432/ogon2025
OPENAI_API_KEY=sk-your-openai-api-key-here
OPENAI_MODEL=gpt-4-turbo-preview
OPENAI_TEMPERATURE=0.7
WEBSOCKET_PORT=3001
WEBSOCKET_PATH=/ws
CORS_ORIGIN=http://localhost:8080
SUPABASE_URL=https://your-project.supabase.co  # Optional
SUPABASE_KEY=your-supabase-anon-key           # Optional

# Frontend environment files (apps/frontend/)
# .env.development - VITE_API_URL=https://dev.api.nltosql.com
# .env.production - VITE_API_URL=https://api.nltosql.com  
# .env.local - For local overrides (gitignored)
```

#### Production Environment Variables (Dokploy)
- `DATABASE_URL` - PostgreSQL connection string
- `OPENAI_API_KEY` - AI service integration
- `POSTGRES_PASSWORD` - Database password
- `REDIS_PASSWORD` - Redis password (production)
- `CORS_ORIGIN` - Frontend URL (must match exactly)
- `VITE_API_URL` - Frontend API URL (build time injection)

#### Application Environment Variables
```bash
# Production Backend
NODE_ENV=production
DATABASE_URL=postgresql://user:pass@host:port/db
WALLET_DATABASE_URL=postgresql://wallet_user:wallet_pass@host:port/wallet_db
PAM_DATABASE_URL=postgresql://pam_user:pam_pass@host:port/pam_db
GAMES_DATABASE_URL=postgresql://games_user:games_pass@host:port/games_db
KYC_DATABASE_URL=postgresql://kyc_user:kyc_pass@host:port/kyc_db
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_KEY=your-supabase-anon-key
PORT=3000
LOG_LEVEL=info
OPENAI_API_KEY=sk-...
CORS_ORIGIN=https://nltosql.com

# Development Backend  
NODE_ENV=development
DATABASE_URL=postgresql://user:pass@host:port/dev_db
WALLET_DATABASE_URL=postgresql://wallet_user:wallet_pass@host:port/wallet_dev_db
PAM_DATABASE_URL=postgresql://pam_user:pam_pass@host:port/pam_dev_db
GAMES_DATABASE_URL=postgresql://games_user:games_pass@host:port/games_dev_db
KYC_DATABASE_URL=postgresql://kyc_user:kyc_pass@host:port/kyc_dev_db
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_KEY=your-supabase-anon-key
CORS_ORIGIN=https://dev.nltosql.com
LOG_LEVEL=debug

# Frontend (build time via Vite)
VITE_API_URL=https://api.nltosql.com       # Production
VITE_API_URL=https://dev.api.nltosql.com   # Development
VITE_API_URL=/api                          # Local (proxy)
```

## Operational Procedures

### Deployment Process
1. **Development**: Feature branch → PR → Code review
2. **CI Validation**: Optional GitHub Actions validation (not required)
3. **Development Deploy**: Push to develop → Auto-deploy via Dokploy
4. **Production Deploy**: Push to main → Auto-deploy via Dokploy
5. **Health Monitoring**: Manual monitoring via Dokploy dashboard

### Important Notes for Dokploy

#### Health Checks Disabled
- Docker HEALTHCHECK instructions removed from Dockerfiles
- Health check routes disabled in backend (`/api/health` endpoints)
- Health check sections removed from docker-compose files
- This resolves container restart loops that occurred in Dokploy

#### CORS Configuration
- Production: `CORS_ORIGIN=https://nltosql.com`
- Development: `CORS_ORIGIN=https://dev.nltosql.com`
- Must match frontend URL exactly (no trailing slash)

#### Frontend Nginx Proxy
- Frontend uses relative `/api` paths for all API calls
- Nginx proxies `/api` requests to `$BACKEND_URL`
- `BACKEND_URL` is set via environment variable at runtime

### Emergency Procedures
1. **Production Issues**: Use Dokploy dashboard to check logs and container status
2. **Immediate Rollback**: Use Dokploy rollback feature
3. **Service Restart**: Use deployment environment controls
4. **Database Backup**: Automated before migrations

### Monitoring & Observability
- **Health Endpoints**: `/api/health`, `/api/health/db`, `/health`
- **Logging**: Structured JSON logs with Fastify
- **Metrics**: Response times, error rates, resource usage
- **Alerts**: Failed deployments, health check failures

### Security Practices
- **Container Security**: Non-root users, minimal base images
- **Network Security**: CORS, CSP headers, rate limiting
- **Vulnerability Management**: Daily scans, automated updates
- **Secrets Management**: GitHub Secrets, no hardcoded credentials

### Performance Optimization
- **Backend**: Connection pooling, Redis caching, structured logging
- **Frontend**: Code splitting, bundle analysis, service worker
- **Database**: Query optimization, connection management
- **Infrastructure**: Container resource limits, health checks

### Troubleshooting Checklist

#### Development Issues
1. Check environment setup: Follow Quick Start section above
2. Verify database: `pg_isready -h localhost -p 5432` and check DATABASE_URL in .env
3. Clear caches: `yarn clean && yarn install`
4. Check logs: `yarn dev` (watch for errors)
5. **TailwindCSS v4 Issues**: Ensure `postcss.config.js` uses `@tailwindcss/postcss` plugin
6. **React 19 Issues**: Ensure @types/react and @types/react-dom are ^19.0.0

#### Production Issues  
1. Health check: Access `/api/health` and `/api/health/db` endpoints
2. View logs: Check deployment environment logs
3. Check metrics: Monitor response times and error rates
4. Rollback if needed: Use GitHub Actions deployment rollback feature

#### CI/CD Issues
1. Check GitHub Actions logs
2. Verify secrets and environment variables
3. Test locally: `yarn lint && yarn typecheck && yarn test`
4. Review deployment status in GitHub environments
5. **Yarn Version Issues**: Ensure Corepack is enabled in CI for Yarn 4.1.1 compatibility
6. **Missing Secrets**: Optional secrets like SLACK_WEBHOOK_URL should be configured conditionally
7. **Supabase Test Failures**: Integration tests automatically skip when `CI=true` to avoid credential requirements

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/node-develop)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/node-develop)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
