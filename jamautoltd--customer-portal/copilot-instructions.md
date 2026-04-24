## customer-portal

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

JAM Auto is a monorepo for a job scheduling and optimization system that manages technician assignments and routes for automotive services (ADAS calibration, key programming, module replacement, diagnostics).

## Project Memory System

The `.cursor/rules/` directory contains a comprehensive knowledge base that serves as persistent memory across development sessions. When working on this codebase:

1. **Start with** `.cursor/rules/project_memory.mdc` - the master index containing:
   - Current project state and recent changes
   - Links to all other memory files
   - Quick reference for common tasks

2. **Key memory files to reference**:
   - `project_context.mdc` - Architecture decisions, conventions, tech stack details
   - `recent_decisions.mdc` - Chronological log of implementation choices
   - `pattern_library.mdc` - Reusable code patterns and examples
   - `known_issues.mdc` - Common problems and their solutions
   - `implementation_history.mdc` - Feature development timeline

3. **Feature-specific rules**:
   - **Scheduler**: `sched-*.mdc` files for orchestration, availability, eligibility, logging
   - **Optimizer**: `optimizer-*.mdc` files for constraints, break items
   - **Testing**: `integration-testing-*.mdc`, `scenario-*.mdc` for test patterns
   - **Database**: `database_schema.mdc` for complete schema reference
   - **Development**: `dev_workflow.mdc` for Task Master workflow

4. **When making changes**:
   - Check relevant rule files before implementing features
   - Follow established patterns from `pattern_library.mdc`
   - Update memory files when discovering new patterns or making significant decisions
   - Refer to `self_improve.mdc` for memory system maintenance guidelines

This memory system ensures consistent development patterns, preserves important decisions, and provides continuous context across all development sessions.

### Architecture

**Monorepo Structure** (managed with pnpm workspaces):
- `apps/web/` - Next.js customer portal (order placement, job tracking)
- `apps/scheduler/` - Node.js/TypeScript scheduling orchestrator
- `apps/optimiser/` - Python/FastAPI route optimization service (Google OR-Tools)
- `simulation/` - Docker Compose environment for local development/testing
- `docs/` - Technical documentation
- `tests/` - Integration and E2E tests

**Key Interactions**:
1. Frontend → Supabase (direct queries with RLS)
2. Supabase webhooks → Edge Function → Scheduler
3. Scheduler → External APIs (Google Maps, One Step GPS) & Optimizer
4. Scheduler → Optimizer → Supabase (job updates)

## Common Commands

### Development
```bash
# Install all dependencies (run from root)
pnpm install

# Run dev servers
pnpm run dev              # Web app (port 3000) - uses workspace filtering
pnpm run dev:scheduler    # Scheduler service - uses workspace filtering  
pnpm run dev:optimiser    # Optimizer service (port 8080)

# Build for production
pnpm run build:web        # Uses pnpm --filter @jamauto/web run build
pnpm run build:scheduler  # Uses pnpm --filter @jamauto/scheduler run build
```

### Testing
```bash
# Run all unit tests
pnpm run test

# Run specific service tests  
pnpm run test:web         # Uses pnpm --filter @jamauto/web run test
pnpm run test:scheduler   # Uses pnpm --filter @jamauto/scheduler run test
pnpm run test:optimiser

# Run integration tests
pnpm run test:integration

# Run E2E tests (requires simulation environment)
pnpm run test:e2e --generate

# Run a single test file
pnpm test -- path/to/test.test.ts

# Run tests in watch mode
pnpm test -- --watch
```

### Integration Test Workflow (Important!)
```bash
# CORRECT order of operations for integration tests:
# 1. Seed baseline data (once per session)
pnpm run db:seed:staging

# 2. For each test scenario - use proper technician count!
pnpm db:seed:staging -- --action scenario --name SCENARIO_NAME --techs N \
  --baseline-metadata tests/integration/.baseline-metadata.json \
  --output-metadata tests/integration/.current-scenario-metadata.json

# 3. Run test with timestamp-coordinated log capture
START_TIME=$(date -u +"%Y-%m-%dT%H:%M:%S.%3NZ")
pnpm jest tests/integration/scheduler/SCENARIO_NAME.test.ts
END_TIME=$(date -u +"%Y-%m-%dT%H:%M:%S.%3NZ")
docker logs test_scheduler --since "$START_TIME" --until "$END_TIME" > "debug/SCENARIO_NAME_scheduler.log" 2>&1

# Note: Use MCP Supabase tool to inspect staging database (nvjdgldtvuhowarpulyl) during debugging
```

### Code Quality
```bash
# Lint all code
pnpm run lint

# Format code
pnpm run format

# Type checking (web app)
pnpm run type-check
```

### Database & Simulation
```bash
# Generate TypeScript types from Supabase
pnpm run db:generate-types

# Seed staging database
pnpm run db:seed:staging

# Clean staging database
pnpm run db:clean:staging

# Start simulation environment
cd simulation && docker-compose up -d

# Interactive E2E test runner
pnpm run e2e:run
```

### Cleanup
```bash
# Remove all node_modules, build artifacts, and lock files
pnpm run clean
```

## High-Level Architecture

### Scheduling Flow
1. **Job Creation**: Orders create jobs with priority (insurance > commercial > residential)
2. **Trigger**: Database webhooks or periodic Cloud Scheduler triggers `/run-replan`
3. **Orchestration** (`runFullReplan` in orchestrator.ts):
   - Fetch relevant jobs (queued, en_route, in_progress, fixed_time)
   - Determine technician availability (default hours - exceptions - locked jobs)
   - Bundle jobs by order_id (except fixed_time jobs)
   - Check equipment eligibility
   - Prepare optimization payload for each day
   - Call optimizer service
   - Process results and update job assignments
4. **Optimization**: Python service uses OR-Tools to solve VRP with constraints

### Key Concepts
- **Job States**: pending → queued → en_route → in_progress → completed
- **Fixed Time Jobs**: Must be scheduled at exact time, bypass normal optimization
- **Job Bundles**: Multiple jobs from same order scheduled together
- **Equipment Requirements**: Jobs need specific equipment; technicians must have it on their van
- **Availability Windows**: Technician working hours minus exceptions and locked jobs
- **Priority Handling**: Higher priority jobs have higher penalties for being unassigned
- **Customer Activation**: Staff-created accounts require email activation (no temporary passwords shown)

### Database Schema
- Core tables: orders, jobs, technicians, addresses, services
- Equipment: van_equipment, equipment_requirements table (unified)
- Availability: technician_default_hours, technician_availability_exceptions
- See `/docs/reference/database_schema.md` for details

### External Integrations
- **Supabase**: Database, Auth, Edge Functions, Webhooks
- **Google Maps**: Distance Matrix API for travel times
- **One Step GPS**: Real-time technician locations
- **Google Cloud**: Cloud Run (services), Cloud Build (CI/CD), Cloud Scheduler

## Development Notes

- All commands should be run from the **root** directory
- Frontend uses Supabase directly (no backend API calls)
- Scheduler is triggered by webhooks, not user actions
- Use simulation environment for backend testing
- Integration tests use staging database with test data

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JAMAutoLtd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
