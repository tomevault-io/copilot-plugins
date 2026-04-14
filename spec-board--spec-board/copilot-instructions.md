## spec-board

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

SpecBoard is a visual dashboard for spec-kit projects with a Kanban board interface for tracking feature development. It uses a **database-first** architecture - all project content (specs, plans, tasks, clarifications) is stored in PostgreSQL.

**Legacy**: The `lib/parser.ts` module exists for parsing markdown files but is no longer used for the main data flow.

## Tech Stack

- **Framework**: Next.js 16 (App Router, Turbopack)
- **Database**: PostgreSQL with Prisma ORM (uses Supabase for cloud deployments)
- **Auth**: Better Auth with OAuth (Google, GitHub)
- **State**: Zustand
- **UI**: Tailwind CSS v4, shadcn/ui components, Lucide icons
- **AI**: Anthropic Claude API via `AIService` (configurable: OpenAI, Anthropic, or any OpenAI-compatible API)
- **Code Execution**: E2B sandbox
- **Package Manager**: pnpm

## Commands

```bash
# Development
pnpm dev              # Start dev server with Turbopack (port 3000)
pnpm dev:all         # Run dev server + worker (if needed)
pnpm build           # Production build
pnpm start           # Start production server

# Database
pnpm db:studio       # Open Prisma Studio
pnpm db:migrate      # Create migration
pnpm db:push         # Push schema to DB
pnpm db:seed         # Seed database
pnpm db:reset        # Reset database (migrate reset --force)

# Testing
pnpm test           # Run tests in watch mode
pnpm test:run       # Run tests once
pnpm test:coverage  # Run tests with coverage

# Linting & Type check
pnpm lint           # Run linter
pnpm tsc --noEmit   # Type check
```

## Architecture

### Database-First

All content is stored in PostgreSQL. The Feature model contains content fields:
- `specContent`, `planContent`, `tasksContent`, `clarificationsContent`
- `researchContent`, `dataModelContent`, `quickstartContent`
- `contractsContent`, `checklistsContent`, `analysisContent`

### Data Flow

```
PostgreSQL → API Routes (/api/project/[name]/data) → Zustand Store → React UI
```

### Workflow Stages

Current feature workflow (4 stages):
| Stage | Description |
|-------|-------------|
| `backlog` | Feature ideas and descriptions |
| `specs` | Spec + Clarifications (merged from specify + clarify) |
| `plan` | Implementation plan with checklist (merged from plan + checklist) |
| `tasks` | Task breakdown + Analysis (final stage, analysis runs automatically) |

### URL Structure

| Route | Purpose |
|-------|---------|
| `/` | Home - project list from database |
| `/projects/:slug` | Kanban board (slug from DB) |
| `/projects/:slug/features/:id` | Feature detail modal |
| `/projects/:slug` | Constitution via Project Info modal |
| `/cloud` | Cloud sync dashboard |
| `/settings` | App settings |

**Important**: URLs use database slugs, not filesystem paths.

## Core Features

### SPECS Stage (New)
The SPECS stage merges the old Specify and Clarify stages into one:
- **Left panel**: Interactive Q&A (ClarificationForm)
- **Right panel**: Document viewer with User Stories
- **Auto-generate**: When transitioning from backlog → specs, both spec AND questions are generated

### Plan Stage (Updated)
The Plan stage now includes checklist:
- When transitioning from specs → plan, both plan AND checklist are generated

### Constitution System
Project-level principles stored in database with version tracking and history. All editing is done via Project Info modal. Key files:
- `src/components/constitution-editor.tsx` - Constitution editing UI (used in modal)
- `src/components/project-info-bubble.tsx` - Project info modal with Constitution tabs

### Project Info Modal
Modal accessible via "Project Info" button on project page showing:
- Project description (editable)
- Constitution principles, version, dates
- Constitution history (version timeline)

### Feature Detail V2
The primary feature detail UI. Located at `src/components/feature-detail-v2/`. This is the ONLY supported feature detail UI.

### Keyboard Navigation
- Full keyboard navigation support with number keys (1-4) to switch between stages
- Navigate features with arrow keys when focus state is active

## Core Patterns

### API Routes

- `/api/projects` - CRUD for projects (name/slug stored in DB)
- `/api/project/:name/data` - Load project data from database
- `/api/browse` - List all projects from database
- `/api/spec-workflow/specify` - Generate spec from description
- `/api/spec-workflow/clarify` - Generate clarification questions
- `/api/spec-workflow/plan` - Generate implementation plan
- `/api/spec-workflow/checklist` - Generate quality checklist
- `/api/spec-workflow/tasks` - Generate task breakdown
- `/api/spec-workflow/analyze` - Analyze consistency
- `/api/checklist` - Toggle checklist items
- `/api/sync/*` - Cloud sync operations
- `/api/auth/*` - Better Auth OAuth

### State Management

Zustand store (`lib/store.ts`) manages:
- `project`: Current loaded project
- `selectedFeature`: Currently selected feature
- `focusState`: Keyboard navigation (column, cardIndex, featureId)

### AI Integration

All AI operations use `AIService` from `src/lib/ai/client.ts`. AI settings stored in database, retrieved via `getAISettings()`.

## Mandatory Policies

### UI Requirements
- **ALWAYS use FeatureDetailV2** (`src/components/feature-detail-v2/`) - the only supported UI
- Never use `?legacy=true` to access old UI

### AI Requirements
- **ALWAYS use real AI** - call actual AI APIs, never mock data
- Use `getAISettings()` to retrieve API keys from database
- Use `AIService` from `src/lib/ai/client.ts` for all AI operations
- If no API key configured, throw an error - do NOT fallback to fake data

## Code Conventions

- **Imports**: Use `@/` alias (e.g., `import from '@/lib/store'`)
- **Components**: `'use client'` directive, PascalCase, `cn()` for conditional classes
- **Types**: Import from `@/types`
- **Styling**: Tailwind CSS v4 with CSS variables (`var(--foreground)`, etc.)

## Testing

Tests co-located with source files:
- `src/lib/*.test.ts`
- `src/app/api/**/*.test.ts`

Run specific test: `pnpm vitest run src/lib/parser.test.ts`
Run single test file: `pnpm test:run path/to/test.test.ts`

## Key Files

| File | Purpose |
|------|---------|
| `src/app/api/project/[name]/data/route.ts` | Main DB-first data endpoint |
| `src/lib/store.ts` | Zustand state management |
| `src/lib/ai/client.ts` | AI service with real implementations |
| `prisma/schema.prisma` | Database schema |
| `src/components/feature-detail-v2/` | Feature detail UI (ONLY supported) |
| `src/components/kanban-board.tsx` | Kanban board with stage transitions |
| `scripts/migrate-stages.ts` | Migration script for stage updates |
| `scripts/worker.ts` | BullMQ worker for background jobs |
| `src/lib/path-utils.ts` | Path validation for security |

## Migration Scripts

When stages change, run migration scripts:
```bash
pnpm tsx scripts/migrate-stages.ts
```

This handles transitions like:
- `specify` → `specs`
- `clarify` → `specs`
- `checklist` → `plan`

## Security

All filesystem operations use `isPathSafe()` from `src/lib/path-utils.ts` to prevent directory traversal attacks. This validation is enforced in all API routes that handle file paths.

XSS prevention: All HTML content is sanitized with DOMPurify before rendering.

## Environment Setup

Copy `.env.example` to `.env` and configure:
- `POSTGRES_PRISMA_URL` - PostgreSQL connection string
- `BETTER_AUTH_SECRET` - Auth secret key
- AI provider settings (API keys for OpenAI, Anthropic, etc.)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spec-board)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/spec-board)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
