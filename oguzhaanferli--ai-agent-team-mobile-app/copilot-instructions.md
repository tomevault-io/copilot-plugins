## ai-agent-team-mobile-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI-driven mobile app development project using a team of 4 specialized AI agents (Team Lead, Designer, Frontend, Backend) that work iteratively with human developer approval at each phase. Documentation is primarily in Turkish.

**Current state:** Architecture and standards are defined but no application code has been scaffolded yet. The repository contains agent role definitions and development templates/standards only.

## Repository Structure

- `agents/` - AI agent role definitions (team_lead, designer, frontend, backend). Each has a `role.md` describing responsibilities and protocols.
- `templates/` - Development standards and architectural guidelines:
  - `design/` - Design tokens (colors, typography, spacing), wireframe standards, component design templates
  - `frontend/` - React Native component standards, state management patterns
  - `backend/` - API standards, database schema conventions, auth strategy
- `operations/` - Operations center for team coordination:
  - `hub/` - Daily coordination (task board, sprint planning, agent status, daily workflow)
  - `workflows/` - Process definitions (feature development, bug fix, code review, deployment, release)
  - `communication/` - Agent interaction protocols (handoff specs, escalation paths, ADR template)
  - `quality/` - Phase gate checklists (design, frontend, backend, integration, release readiness)
  - `tracking/` - Progress tracking (sprint log, milestone tracker, decision log with 3 existing ADRs)
- `context/` - Project requirements and specifications:
  - `project-requirements.md` - Product requirements document template
  - `user-stories.md` - User story backlog (auth stories pre-populated)
  - `feature-specs/` - Per-feature specification templates
  - `technical-constraints.md` - Aggregated tech decisions and constraints
  - `glossary.md` - Turkish↔English term glossary
- `deliverables/` - Completed outputs (to be populated)
- `tests/` - Test files (to be populated)

## Target Technology Stack

### Frontend (Mobile)
- **React Native** with TypeScript strict mode
- **Zustand** for global state (auth, user, settings) with AsyncStorage persistence
- **React Query/TanStack Query** for server state (API data, pagination)
- **React Hook Form** for forms
- **Axios** for HTTP
- **React Navigation** for routing
- **Nativewind** + StyleSheet for styling
- **Jest** + **Detox** for testing

### Backend
- **Node.js + Express.js** (NestJS for enterprise)
- **PostgreSQL 15+** with **Prisma** ORM
- **JWT** auth (RS256) - access token 1hr, refresh token 14 days
- **Zod** for validation
- **Redis** for caching
- **Winston** for logging
- **Swagger/OpenAPI** for docs

## Key Conventions

### Naming
| Element | Convention | Example |
|---------|-----------|---------|
| Components | PascalCase | `UserProfile.tsx` |
| Functions | camelCase | `getUserData` |
| Constants | UPPER_SNAKE_CASE | `API_BASE_URL` |
| Files | PascalCase | `UserProfile.tsx` |
| Folders | kebab-case | `user-profile/` |
| DB tables | snake_case, plural | `users`, `order_items` |
| DB columns | snake_case | `created_at`, `is_active` |
| API URLs | kebab-case, versioned | `/api/v1/user-profiles` |

### Frontend Component Structure
Each component gets its own directory:
```
ComponentName/
├── index.ts
├── ComponentName.tsx
├── ComponentName.styles.ts
├── ComponentName.types.ts
└── ComponentName.test.tsx
```

Components should use `memo()`, include `testID` props, and have accessibility attributes.

### State Management Pattern
- **UI state** → `useState`
- **Form state** → React Hook Form
- **Global client state** (auth, cart) → Zustand with persist middleware
- **Server/async data** → React Query with 5min stale time

### API Response Format
All endpoints follow this structure:
```json
{
  "success": true,
  "data": { },
  "meta": { "pagination": { "page": 1, "limit": 10, "total": 100 } }
}
```
Errors use standard codes: `VALIDATION_ERROR`, `UNAUTHORIZED`, `FORBIDDEN`, `NOT_FOUND`, `RATE_LIMIT_EXCEEDED`.

### Database
- Primary keys: UUID (`@default(uuid())`)
- Soft deletes via `deleted_at` column
- All models include `created_at`, `updated_at`
- Foreign keys named `{entity}_id`
- Booleans prefixed with `is_` or `has_`

## Development Workflow

1. One agent focuses on one area per iteration
2. Human developer approves each phase before proceeding
3. Team Lead agent reviews all outputs from other agents
4. All decisions must be documented
5. Quality gates must pass before phase transitions (see `operations/quality/`)
6. Feature development follows: Design → Backend → Frontend → Integration (see `operations/workflows/feature-development.md`)
7. Agent handoffs follow standardized protocols (see `operations/communication/handoff-protocols.md`)
8. Task progress tracked on Kanban board (see `operations/hub/task-board.md`)

## Performance Targets

- **Frontend:** 60 FPS animations, <3s TTI, <10MB bundle, >70% test coverage
- **Backend:** <200ms API response (p95), <50ms DB queries (avg), >99.9% uptime
- **Accessibility:** WCAG compliant, 44x44px minimum touch targets, 4.5:1 color contrast

## Design System

Primary color: `#2196F3` (blue). 8px grid spacing system. Font family: Inter (UI) + JetBrains Mono (code). Breakpoints: 320px (mobile min), 768px (tablet), 1024px+ (desktop).

---
> Source: [oguzhaanferli/ai-agent-team-mobile-app](https://github.com/oguzhaanferli/ai-agent-team-mobile-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
