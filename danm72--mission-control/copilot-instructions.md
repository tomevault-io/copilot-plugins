## mission-control

> SaaS coordination layer for AI agent squads. Users design custom squads through conversation, then their local OpenClaw agent bootstraps the entire team automatically.

# Mission Control

SaaS coordination layer for AI agent squads. Users design custom squads through conversation, then their local OpenClaw agent bootstraps the entire team automatically.

## Tech Stack

| Layer | Technology |
|-------|------------|
| Monorepo | pnpm workspaces + Turborepo |
| Frontend | Next.js 15, React 19, TypeScript |
| Styling | Tailwind CSS, shadcn/ui components |
| Database | Supabase (PostgreSQL + Realtime) |
| Auth | Supabase Auth (email/password) |
| Hosting | Vercel |
| Rate Limiting | Upstash Redis |
| AI | Anthropic Claude via AI SDK v6 |

## Implementation Status

| Phase | Name | Status |
|-------|------|--------|
| 00 | Testing Foundation | Complete |
| 01 | Foundation | Complete |
| 02 | Dashboard Core | In Progress (~35%) |
| 03 | Skills Architecture | In Progress (~85%) |
| 04 | Integration | Pending |
| 05 | Polish & Optimization | Pending |

**Current Wave:** Implementing Phase 02 (Kanban board, task panels) and Phase 03 (API endpoints)

## Directory Structure

```
mission-control/
├── apps/
│   └── web/                     # Next.js 15 dashboard
│       └── src/
│           ├── app/             # App Router
│           │   ├── (auth)/      # Login, signup
│           │   ├── (dashboard)/ # Protected routes
│           │   ├── api/         # API routes
│           │   └── onboarding/  # Squad design
│           ├── components/      # Atomic Design
│           │   ├── atoms/       # Text, Button, Icon, Avatar, Badge, Skeleton, StatusDot
│           │   ├── molecules/   # Clock, StatBadge
│           │   ├── organisms/   # Header, AgentSidebar, LiveFeed
│           │   ├── templates/   # DashboardLayout
│           │   └── onboarding/  # SquadChat, SquadPreview, AgentCard
│           └── lib/             # Utilities
│               ├── auth/        # API key verification
│               └── supabase/    # Client utilities
├── packages/
│   ├── database/                # @mission-control/database
│   │   ├── src/                 # Client, types, test utils
│   │   └── supabase/migrations/ # SQL migrations
│   └── shared/                  # Shared utilities (placeholder)
├── skills/                      # OpenClaw skills
│   ├── mission-control-setup/   # Bootstrap wizard
│   └── mission-control/         # Runtime API client
├── agents/                      # Agent templates
│   ├── lead/SOUL.md
│   ├── writer/SOUL.md
│   └── social/SOUL.md
├── docs/
│   ├── ARCHITECTURE.md          # System architecture
│   ├── API.md                   # API reference
│   ├── SETUP.md                 # Setup guide
│   └── plans/                   # Implementation plans
└── ralph/                       # PRD and progress tracking
    ├── prd/                     # Phase specifications
    └── progress/                # Progress logs
```

## Development Commands

```bash
# Install dependencies
pnpm install

# Start all apps
pnpm dev

# Run only web app
pnpm --filter web dev

# Build
pnpm build

# Lint
pnpm lint

# Format
pnpm format

# Type check
pnpm typecheck

# Run tests
pnpm test

# Run tests with coverage
pnpm --filter web test:coverage

# E2E tests
pnpm --filter web test:e2e
```

## Database Commands

```bash
# Apply migrations
pnpm --filter @mission-control/database db:push

# Generate types from schema
pnpm --filter @mission-control/database db:types

# Reset local database
pnpm --filter @mission-control/database db:reset

# Generate migration diff
pnpm --filter @mission-control/database db:diff <name>
```

## Database Schema

**13 tables with full RLS:**

| Table | Purpose |
|-------|---------|
| `squads` | Multi-tenant workspaces (API key, owner) |
| `agent_specs` | Agent definitions (SOUL.md source) |
| `agents` | Runtime agent instances |
| `tasks` | Kanban tasks |
| `task_assignees` | Many-to-many assignments |
| `messages` | Task comments |
| `documents` | Deliverables |
| `activities` | Activity feed |
| `notifications` | @mention notifications |
| `subscriptions` | Task subscriptions |
| `squad_chat` | Team chat |
| `direct_messages` | Human-agent 1:1 |
| `watch_items` | Monitored items |

**Enums:** `agent_status`, `task_status`, `task_priority`, `document_type`, `activity_type`

## API Patterns

### Authentication

```typescript
// Agent API routes use verifyApiKeyWithAgent
const { squad, agent, supabase, error } = await verifyApiKeyWithAgent(request)
if (error) return error

// RLS context is automatically set
```

### Response Format

```typescript
// Success
return NextResponse.json({
  data: result,
  meta: { count: result.length, timestamp: new Date().toISOString() }
})

// Error
return NextResponse.json({
  error: 'Human-readable message',
  code: 'MACHINE_READABLE_CODE'
}, { status: 400 })
```

### Rate Limits

| Endpoint | Limit |
|----------|-------|
| `/api/heartbeat` | 10/min |
| `/api/tasks*` | 30/min |
| Default | 60/min |

## Component Architecture

### Atomic Design

```
atoms/       → Text, Button, Icon, Avatar, Badge, Skeleton, StatusDot
molecules/   → Clock, StatBadge
organisms/   → Header, AgentSidebar, LiveFeed, KanbanBoard (planned)
templates/   → DashboardLayout
```

### Patterns

- All atoms use `forwardRef`
- `cn()` utility for Tailwind class merging
- Design tokens: `text-text`, `bg-background-elevated`, `status-active`
- Container/View separation for complex organisms
- Compound components for flexibility

## Code Style

### TypeScript

- Strict mode enabled
- No `any` - use `unknown` and narrow
- Prefer `interface` over `type` for objects
- Use `as const` for literal types

### React

- Functional components only
- Server components by default, `'use client'` when needed
- Co-locate component, test, types
- Compound component pattern for complex UI

### Naming

| Type | Convention | Example |
|------|------------|---------|
| Components | PascalCase | `TaskCard.tsx` |
| Hooks | `use` prefix | `useTaskQuery.ts` |
| Utils | camelCase | `formatDate.ts` |
| Types | PascalCase | `type TaskStatus` |
| Constants | SCREAMING_SNAKE | `MAX_RETRY_COUNT` |

### Imports Order

1. React/Next
2. External packages
3. Internal packages (`@mission-control/*`)
4. Relative imports

## Git Conventions

### Commit Messages

```
type(scope): brief description

feat(api): add heartbeat endpoint
fix(dashboard): resolve task card overflow
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

### Branch Naming

```
feat/short-description
fix/issue-number-description
chore/task-description
```

## Environment Variables

```bash
# Required
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key

# Rate limiting
UPSTASH_REDIS_REST_URL=your-upstash-url
UPSTASH_REDIS_REST_TOKEN=your-upstash-token

# AI (onboarding)
ANTHROPIC_API_KEY=your-anthropic-key
```

## Testing

### Coverage Targets

| Code Type | Target |
|-----------|--------|
| Business Logic | 90% |
| React Components | 80% |
| API Routes | 85% |
| Utilities | 95% |

### Test Organization

- Component tests co-located: `ComponentName.test.tsx`
- Integration tests in `__tests__/`
- E2E tests in `tests/e2e/`

## Key Files

### PRD Documents

- `ralph/prd/phase-00-testing-foundation.md`
- `ralph/prd/phase-01-foundation.md`
- `ralph/prd/phase-02-dashboard-core.md`
- `ralph/prd/phase-03-skills.md`
- `ralph/prd/phase-04-integration.md`
- `ralph/prd/phase-05-polish.md`

### Progress Tracking

- `ralph/HANDOVER.md` - Current orchestration state
- `ralph/progress/phase-*.txt` - Detailed progress logs
- `ralph/progress/phase-*.done` - Phase completion markers

### Documentation

- `docs/ARCHITECTURE.md` - System architecture
- `docs/API.md` - API reference
- `docs/SETUP.md` - Setup guide
- `docs/testing.md` - Testing guide

## Architecture Decisions

1. **Monorepo** - Shared code, atomic deployments
2. **Server Components First** - Better performance
3. **Supabase RLS** - Multi-tenant data isolation
4. **API Keys per Agent** - Independent authentication
5. **Pull-Based Notifications** - Agents fetch on heartbeat
6. **SOUL.md Sync** - Dashboard is source of truth
7. **Optimistic UI** - Immediate feedback
8. **Atomic Design** - Composable components

---
> Source: [Danm72/mission-control](https://github.com/Danm72/mission-control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
