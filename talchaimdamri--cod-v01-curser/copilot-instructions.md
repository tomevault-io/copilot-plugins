## cod-v01-curser

> A lightweight, modular workspace application for creating and managing document processing chains with AI agents. Built with React 18 + Vite frontend, Node.js 20 + Fastify backend, PostgreSQL 16 database, and Docker containerization.

# Chain Workspace Application - Development Guide

## Project Overview

A lightweight, modular workspace application for creating and managing document processing chains with AI agents. Built with React 18 + Vite frontend, Node.js 20 + Fastify backend, PostgreSQL 16 database, and Docker containerization.

## Architecture Patterns

- **Event-sourced state**: Immutable event log with undo/redo capabilities
- **Single-process launch**: Optimized for rapid iteration, horizontally scalable later
- **TDD workflow**: Test-first development with specialized sub-agents
- **Schema-driven development**: Zod schemas as single source of truth for data contracts
- **Memory Bank**: Long-term knowledge base maintained by commit-bot agent

## Key Technologies

- **Frontend**: React 18, Vite, Tailwind CSS, SVG rendering
- **Backend**: Node.js 20, TypeScript, Fastify, Zod validation
- **Database**: PostgreSQL 16, Prisma ORM
- **Testing**: Vitest (unit), Supertest (integration), Playwright (E2E)
- **Containerization**: Docker + Compose

## Development Commands

### Environment Setup

```bash
# Start local development environment
docker-compose -f docker-compose.dev.yml up -d

# Install dependencies
npm install

# Run database migrations
npm run db:migrate

# Seed development data
npm run db:seed
```

### Development Workflow

```bash
# Start frontend development server
npm run dev:ui

# Start backend development server
npm run dev:api

# Run both frontend and backend concurrently
npm run dev

# Build for production
npm run build
```

### Testing Commands

```bash
# Run unit tests
npm run test:unit

# Run integration tests (requires running database)
npm run test:integration

# Run E2E tests
npm run test:e2e

# Run all tests
npm test

# Run tests with coverage
npm run test:coverage

# Run tests in watch mode
npm run test:watch
```

### Database Operations

```bash
# Create new migration
npm run db:migrate:create

# Deploy migrations
npm run db:migrate:deploy

# Reset database (development only)
npm run db:reset

# Open database GUI (Adminer)
open http://localhost:8081
```

### Code Quality

```bash
# Run linter
npm run lint

# Fix linting issues
npm run lint:fix

# Run type checking
npm run typecheck

# Format code
npm run format
```

### Docker Operations

```bash
# Build all containers
docker-compose build

# Start services in background
docker-compose up -d

# View logs
docker-compose logs -f

# Stop all services
docker-compose down

# Clean up volumes (development reset)
docker-compose down -v
```

## Project Structure

```
COD_V01/
├── .claude/agents/          # Sub-agent definitions
├── .memri/                  # Memory Bank knowledge files
├── src/                     # React frontend source
│   ├── components/          # Reusable UI components
│   ├── pages/              # Route components
│   └── lib/                # Utilities and hooks
├── api/                     # Node.js backend source
│   ├── routes/             # Fastify route handlers
│   ├── services/           # Business logic
│   └── db/                 # Database utilities
├── schemas/                 # Shared Zod schemas
├── tests/                   # Test suites
│   ├── unit/               # Unit tests
│   ├── integration/        # Integration tests
│   └── e2e/                # End-to-end tests
├── migrations/              # Database schema migrations
├── docker/                  # Container configurations
└── docs/                   # Project documentation
```

## Key File Patterns

- Components: `src/components/**/*.tsx`
- API routes: `api/routes/**/*.ts`
- Schemas: `schemas/**/*.ts`
- Tests: `tests/**/*.test.ts`
- Migrations: `migrations/**/*.sql`

## Sub-Agent Workflow

1. **test-runner** - Creates failing tests first (TDD)
2. **schema-keeper** - Defines/updates data contracts
3. **ui-developer** - Implements React components
4. **backend-developer** - Implements API endpoints
5. **commit-bot** - Commits code, runs tests, updates Memory Bank

## Task-Based Commit Format

All commits should reference Task Master task numbers for traceability:

### Commit Message Format

```bash
# Task-specific commits
feat(task-3): implement core Zod schemas for API validation
fix(task-5): resolve canvas pan/zoom boundary calculations
test(task-4): add integration tests for events API endpoints
refactor(task-11): optimize event sourcing performance

# Setup and maintenance commits
feat(setup): integrate Task Master AI for TDD workflow management
docs(memory): update Memory Bank with task completion progress
ci(deps): update dependencies for security patches
```

### Task Completion Commits

When completing tasks, use the commit-bot agent with this format:

```bash
feat(task-X): complete [task title]

- ✅ [subtask 1 description]
- ✅ [subtask 2 description]
- ✅ All tests passing (unit/integration/e2e)
- ✅ Memory Bank updated with progress

Closes task-X
```

### Sub-Agent Commit Guidelines

- **test-runner**: `test(task-X): add [test type] tests for [feature]`
- **schema-keeper**: `feat(task-X): define [schema name] with validation rules`
- **ui-developer**: `feat(task-X): implement [component name] with [key features]`
- **backend-developer**: `feat(task-X): add [endpoint/service] with [functionality]`
- **commit-bot**: Always includes Memory Bank updates and task references

## Memory Bank Files

- `projectbrief.md` - High-level project description (read-only)
- `productContext.md` - Feature requirements and business context
- `systemPatterns.md` - Architecture patterns and conventions
- `techContext.md` - Technology decisions and dependencies
- `activeContext.md` - Current development decisions
- `progress.md` - Development progress tracking

## Quick Start

1. `docker-compose -f docker-compose.dev.yml up -d` - Start database
2. `npm install` - Install dependencies
3. `npm run db:migrate` - Set up database schema
4. `npm run dev` - Start development servers
5. Open http://localhost:4000 for UI, http://localhost:4001 for API

## Environment Variables

```bash
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/chainworkspace

# API Keys (development)
OPENAI_API_KEY=your_openai_key_here

# JWT Secret
JWT_SECRET=your_jwt_secret_here

# Environment
NODE_ENV=development
```

## Ports

- Frontend (Vite): 4000
- Backend (Fastify): 4001
- Database (PostgreSQL): 5435
- Adminer (DB GUI): 8081

## Task Master AI Instructions

**Import Task Master's development workflow commands and guidelines, treat as if import is in the main CLAUDE.md file.**
@./.taskmaster/CLAUDE.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/talchaimdamri) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
