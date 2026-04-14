## clem-ai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Clem AI is an AI-powered insurance claims processing system built as a monorepo with:

- **Backend**: NestJS API with modular architecture (Prisma ORM, PostgreSQL)
- **Frontend**: Next.js 15 with App Router, React 19, and shadcn/ui components
- **Microservices**: Python FastAPI services for AI operations and ETL
- **Shared Packages**: Reusable TypeScript types, utilities, and configurations
- **Package Manager**: pnpm with workspace support
- **AI Integration**: LangChain/LangGraph with Google Gemini for claims processing

## Repository Structure

```plaintext
clem-ai/
├── apps/
│   ├── backend/              # NestJS API (port 3000)
│   └── frontend/             # Next.js dashboard (port 3001)
├── services/
│   ├── ai-service/           # Python FastAPI AI service (port 8000)
│   └── etl-service/          # Python ETL service
├── packages/
│   ├── shared-types/         # Common TypeScript types
│   ├── shared-utils/         # Utility functions
│   ├── eslint-config/        # Shared ESLint configuration
│   └── tsconfig/             # Base TypeScript configurations
├── config/                    # Configuration files (docker-compose, linters, Makefile)
├── docs/                      # Documentation
├── infra/                     # Infrastructure and Kubernetes manifests
└── .github/                   # CI/CD workflows and templates
```

## Architecture

### Backend (NestJS)

**Location**: `apps/backend/`

**Modular Structure** (`src/modules/`):
- **Claims Module**: Core claims processing with LangGraph AI agent workflow
  - Entry point: `ClaimsService.processNewClaim()`
  - Agent workflow: triage node → recommend node
  - State streaming: Agent streams through nodes, final state reconstructed from chunks
  - WebSocket broadcasting: `ClaimsGateway` emits `newClaim` and `claimUpdate` events
- **Audit Module**: Immutable audit logging for all claim operations with actor tracking
- **Email Module**: Email notifications using Handlebars templates
- **Storage Module**: MinIO S3-compatible object storage for documents/photos
- **ETL Module**: Data import/export and transformation operations

**Database**: Prisma ORM with PostgreSQL (`prisma/schema.prisma`)
- Comprehensive Claim model with 50+ fields including AI-generated fields
- Photo/Document models with S3 references
- AuditLog, Communication, and ETLJob models

**Real-time**: Socket.IO WebSocket gateway for live claim updates to dashboard clients

**Key Implementation**: LangGraph agent with structured output using Zod schemas (`claims.state.ts`)

### Frontend (Next.js)

**Location**: `apps/frontend/`

**App Router Structure** (route groups for layout):
- `/(dashboard)/page.tsx` - Main dashboard with KPIs and claims queue
- `/(dashboard)/claims/page.tsx` - Claims list with filtering
- `/(dashboard)/claims/[id]/page.tsx` - Claim detail view
- `/(dashboard)/new-claim/page.tsx` - Claim submission form

**Key Components**:
- `components/claims/` - Claim-specific UI (status badges, filters, damage assessment)
- `components/dashboard/` - Metrics, charts, KPI cards (Recharts)
- `components/layout/` - Sidebar and header navigation

**Form Validation**: React Hook Form + Zod schemas
**Real-time**: Socket.IO client for live updates

### Python Microservices

**AI Service** (`services/ai-service/`, port 8000):
- `/assess-damage` - Damage assessment from photos/descriptions
- `/calculate-settlement` - Settlement calculations
- `/check-policy-eligibility` - Policy coverage validation
- `/assess-vehicle-value` - Vehicle valuation

**ETL Service** (`services/etl-service/`):
- FastAPI with SQLAlchemy and Pydantic
- Data validation, deduplication, survivorship rules
- Rich CLI and Streamlit dashboard

## Development Commands

### Setup
```bash
# Install all dependencies (from root)
pnpm install

# Start PostgreSQL
docker-compose up -d postgres

# Generate Prisma client and run migrations
cd apps/backend
pnpm run postinstall
npx prisma migrate dev
```

### Running Services
```bash
# All services with Docker Compose (recommended)
docker-compose up -d

# Individual services (from root)
pnpm dev                              # Backend + Frontend together
pnpm --filter backend start:dev       # Backend only
pnpm --filter frontend dev            # Frontend only

# AI Service (from services/ai-service/)
uvicorn src.main:app --reload --port 8000

# ETL Service (from services/etl-service/)
python -m src.etl.api.main
```

### Database
```bash
# From apps/backend/
npx prisma studio                     # Open GUI
npx prisma migrate dev --name <name>  # Create migration
npx prisma generate                   # Regenerate client
```

### Testing
```bash
# Root level
pnpm test                             # All workspaces
pnpm test:backend                     # Backend only
pnpm test:frontend                    # Frontend only

# Backend (from apps/backend/)
pnpm test                             # Unit tests
pnpm run test:watch                   # Watch mode
pnpm run test:e2e                     # E2E tests

# Python services
cd services/ai-service && pytest tests/
```

### Code Quality
```bash
# Root level (uses config/ directory)
pnpm lint                             # Run ESLint
pnpm lint:fix                         # Fix ESLint issues
pnpm format                           # Format with Prettier
pnpm format:check                     # Check formatting
```

### Building
```bash
# From root
pnpm build                            # All workspaces

# Individual (from respective directories)
pnpm run build
```

## AI Claims Processing Flow

1. **Claim Submission**: POST `/api/v1/claims` with claim data
2. **State Initialization**: `ClaimsService.processNewClaim()` initializes agent state with claim input
3. **Agent Streaming**: LangGraph agent streams through workflow nodes
   - **Triage Node**: Analyzes severity, fraud indicators, initial assessment (structured output via Zod)
   - **Recommend Node**: Calculates settlement amount with justification
4. **State Reconstruction**: Final state built from stream chunks (see `claims.service.ts:76-83`)
5. **Persistence**: Claim saved to database with all AI-generated fields
6. **Broadcasting**: WebSocket emits `newClaim` event to all connected dashboard clients
7. **Audit Logging**: All operations logged with actor tracking

## Key Implementation Details

### LangGraph Agent State Management
- State schema defined in `claims.state.ts` using Zod
- Agent uses `withStructuredOutput()` for type-safe AI responses
- Stream chunks merged to reconstruct final state
- Both triage result and settlement recommendation are structured objects

### WebSocket Integration
- Backend: `ClaimsGateway` in claims module
- Events: `newClaim`, `claimUpdate`
- CORS configured for localhost:3000-3002 (adjust for production)

### Module Architecture
- Each backend module is self-contained with service, controller, DTOs
- Services injected via NestJS dependency injection
- Prisma service shared across modules for database access

## Environment Variables

### Backend
- `DATABASE_URL` - PostgreSQL connection (default: `postgresql://devuser:devpassword@localhost:5432/clem_ai_db`)
- `GOOGLE_API_KEY` - Google Gemini API key (required for AI)
- `PORT` - Optional, defaults to 3000
- `MINIO_ENDPOINT`, `MINIO_ACCESS_KEY`, `MINIO_SECRET_KEY` - S3 storage

### Frontend
- `NEXT_PUBLIC_API_URL` - Backend API URL (default: `http://localhost:3000`)

### Python Services
- `GOOGLE_API_KEY` - For AI operations
- `DATABASE_URL` - PostgreSQL connection

See `.env.example` and `.env.production.example` for complete examples.

## Service Ports

- Backend (NestJS): 3000
- Frontend (Next.js): 3001 (Docker), 3000 (dev)
- AI Service (FastAPI): 8000
- ETL Service: 8001
- PostgreSQL: 5432
- MinIO: 9000 (API), 9001 (Console)

## Configuration Files

All configuration files are in `config/`:
- `docker-compose.dev.yml` / `docker-compose.prod.yml` - Docker configurations
- `eslint.config.mjs` - ESLint rules
- `.prettierrc` / `.prettierignore` - Prettier configuration
- `.lintstagedrc.json` - Pre-commit hooks
- `.editorconfig` - Editor settings
- `Makefile` - Common development tasks

## Important Notes

- Prisma client must be regenerated after schema changes: `npx prisma migrate dev` or `npx prisma generate`
- Backend modules follow NestJS modular architecture pattern
- Frontend uses route groups `(dashboard)` for consistent layout
- LangGraph agent workflow is defined in `ClaimsService.buildAgent()`
- AI operations distributed between NestJS backend (LangGraph) and Python FastAPI service (advanced ML)
- WebSocket CORS configured for localhost development (restrict in production)
- All claims operations generate audit log entries
- ETL service handles bulk data imports with validation and deduplication

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/khvni)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/khvni)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
