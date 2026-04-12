## developpment-app

> You are a Principal Software Architect and Full-Stack Engineer working on "Sovereign AI," a production-grade multi-tenant AI-native IDE and development platform built with Next.js 15, Drizzle ORM, Clerk B2B auth, and the Portkey AI gateway.

# Sovereign AI — Platform Documentation

## ROLE
You are a Principal Software Architect and Full-Stack Engineer working on "Sovereign AI," a production-grade multi-tenant AI-native IDE and development platform built with Next.js 15, Drizzle ORM, Clerk B2B auth, and the Portkey AI gateway.

---

## ARCHITECTURAL STACK

### Frontend & Framework
- **Framework**: Next.js 15 (App Router) with React Server Components (RSC) and Server Actions
- **Styling**: Tailwind CSS v3 + Shadcn UI — **dark-first theme** (slate-950 background, cyan primary accent)
- **Editor**: Monaco Editor (`@monaco-editor/react`) with Virtual File System (VFS)
- **Drag & Drop**: `@dnd-kit/sortable` for Kanban board
- **State**: Server Components + optimistic UI via React `useOptimistic`

### Backend & Database
- **Database**: PostgreSQL (Neon/Supabase) with pgvector for RAG
- **ORM**: Drizzle ORM (serverless-optimized, type-safe schema inference)
- **Auth**: Clerk B2B (multi-tenant Organizations, JWT tokens)
- **Async Queue**: BullMQ + Upstash Redis (heavy media generation)

### AI & Compute
- **AI Gateway**: Portkey (unified provider API, cost tracking, caching)
  - Entry point: `src/lib/ai/portkey.ts` → `chatCompletion()` / `streamChatCompletion()`
  - Every request saves prompt + completion text to `ai_logs` table
- **Code Sandbox**: E2B Code Interpreter SDK v1.0.4
  - `Sandbox.create()` → `sandbox.runCode(code)` → `sandbox.kill()`
  - API: `src/app/api/execute/route.ts`

### Integrations
- **Wrike**: Bi-directional sync (outgoing Server Actions + incoming webhooks)
- **Vercel**: REST API for programmatic deployments
- **GitHub**: GitHub App model (not OAuth) for repository access

---

## PRODUCTION READINESS STATUS

### ✅ Completed (this sprint)

| Area | What was done |
|------|--------------|
| **Agent Prompts Bug Fix** | `logAIUsage()` now saves `prompt` + `completion` text to DB |
| **AI Logs Page** | `/dashboard/ai-logs` — full interaction history with expandable rows |
| **Dashboard Real Metrics** | Replaced hardcoded `'0'`/`'$0.00'` with live `aiLogs.getTenantCostSummary()` |
| **AI Nav Item** | "AI Logs" link added to sidebar |
| **AI Server Action** | `sendAIMessage()` in `src/lib/actions/ai.ts` |
| **E2B Code Execution** | Re-enabled using `Sandbox.create()` API |
| **Dark Professional UI** | Dark-first theme (slate-950 bg, cyan accents, no childish styling) |
| **Security Headers** | `X-Frame-Options`, `X-Content-Type-Options`, HSTS, Referrer-Policy, Permissions-Policy |
| **Secrets Scrubbed** | `.env.example` now contains only placeholder values |
| **Error Boundaries** | Root + dashboard-scoped `error.tsx` files |
| **Loading States** | Skeleton loaders for dashboard, analytics, ai-logs, projects |
| **ESLint Config** | ESLint 9 flat config (`eslint.config.mjs`) |
| **Prettier Config** | `.prettierrc` (singleQuote, tabWidth 2, printWidth 100) |
| **Test Infrastructure** | Vitest + Playwright + Testing Library installed and configured |
| **65 Unit/Integration Tests** | All passing — see Test Suite section below |
| **CI/CD Pipelines** | GitHub Actions: `ci.yml` (lint+test+build) + `e2e.yml` (Playwright) |

### ⬜ Remaining Backlog

- [ ] pgvector RAG implementation
- [ ] BullMQ worker for media generation
- [ ] GitHub App integration (repo browser)
- [ ] Vercel deployment pipeline UI
- [ ] Real-time updates (WebSocket / SSE for Kanban)
- [ ] E2E tests (require running dev server + seeded DB)

---

## DATABASE SCHEMA

Core tables in `src/lib/db/schema.ts`:

| Table | Key Columns | Notes |
|-------|-------------|-------|
| `organizations` | `id`, `clerkOrgId`, `name`, `slug` | Tenant root, synced via Clerk webhook |
| `projects` | `id`, `tenantId`, `name`, `vfsState` | JSONB VFS state |
| `tasks` | `id`, `tenantId`, `projectId`, `status`, `priority`, `aiContext` | JSONB AI context |
| `ai_logs` | `id`, `tenantId`, `model`, `prompt`, `completion`, `cost`, `totalTokens` | Full prompt + completion saved |
| `files` | `id`, `tenantId`, `projectId`, `path`, `content` | Virtual file system |
| `deployments` | `id`, `tenantId`, `projectId`, `vercelDeploymentId`, `status` | Vercel deploy tracking |
| `wrike_sync_state` | `taskId`, `wrikeTaskId`, `lastSyncedAt` | Bi-directional sync metadata |

**Relationships:**
```
organizations (1) → (N) projects
organizations (1) → (N) tasks
projects (1) → (N) files
projects (1) → (N) deployments
projects (1) → (N) ai_logs
tasks (1) → (1) wrike_sync_state (optional)
```

---

## CODE ORGANIZATION

```
src/
├── app/
│   ├── (auth)/                    # Clerk sign-in / sign-up
│   ├── (dashboard)/               # Protected routes
│   │   ├── layout.tsx             # Sidebar + header shell
│   │   ├── error.tsx              # Dashboard error boundary
│   │   └── dashboard/
│   │       ├── page.tsx           # Stats overview (real DB data)
│   │       ├── loading.tsx        # Skeleton
│   │       ├── ai-logs/
│   │       │   ├── page.tsx       # AI interaction history (RSC)
│   │       │   └── loading.tsx
│   │       ├── analytics/
│   │       │   └── loading.tsx
│   │       └── projects/
│   │           └── loading.tsx
│   ├── api/
│   │   ├── execute/route.ts       # E2B code execution
│   │   ├── analytics/cost-history/route.ts
│   │   ├── files/route.ts
│   │   └── webhooks/
│   │       ├── clerk/route.ts     # Org lifecycle (Svix)
│   │       └── wrike/route.ts     # Task sync (HMAC)
│   ├── error.tsx                  # Root error boundary
│   ├── globals.css                # Dark-first CSS variables
│   └── layout.tsx                 # Root layout (dark class)
├── components/
│   ├── ui/                        # Shadcn primitives
│   ├── analytics/
│   │   └── ai-logs-table.tsx      # Expandable logs table (Dialog detail view)
│   ├── editor/                    # Monaco Editor components
│   └── kanban/
│       └── task-card.tsx          # Shows AI context badge + priority colors
├── lib/
│   ├── ai/
│   │   └── portkey.ts             # chatCompletion(), logAIUsage(), cost calc
│   ├── actions/
│   │   ├── ai.ts                  # sendAIMessage() server action
│   │   ├── projects.ts            # createProject(), archiveProject()
│   │   └── tasks.ts               # createTask(), moveTask(), deleteTask()
│   ├── db/
│   │   ├── schema.ts              # Drizzle table definitions
│   │   ├── dal.ts                 # Data Access Layer (all DB queries)
│   │   └── index.ts               # DB connection (throws if no DATABASE_URL)
│   └── integrations/
│       └── wrike.ts               # Wrike API client + HMAC verification
└── test/
    ├── setup.ts                   # Global mocks (next/cache, next/navigation, clerk)
    ├── unit/
    │   ├── ai/portkey.test.ts
    │   ├── actions/ai.test.ts
    │   ├── actions/projects.test.ts
    │   └── actions/tasks.test.ts
    ├── integration/
    │   ├── api/execute.test.ts
    │   ├── api/analytics.test.ts
    │   └── webhooks/clerk.test.ts
    ├── components/
    │   ├── kanban/task-card.test.tsx
    │   └── analytics/ai-logs-table.test.tsx
    └── e2e/
        ├── auth.spec.ts
        ├── dashboard.spec.ts
        └── projects.spec.ts
```

---

## AI GATEWAY USAGE

All AI calls go through `src/lib/ai/portkey.ts`. Critical contract:

```typescript
// Every call automatically:
// 1. Routes to the right provider via Portkey
// 2. Saves full prompt + completion text to ai_logs table
// 3. Tracks token counts and USD cost

const response = await chatCompletion({
  messages: [{ role: 'user', content: 'Hello' }],
  model: 'gpt-4o-mini',          // or 'claude-3-5-sonnet', etc.
  temperature: 0.7,
}, {
  projectId: 'uuid',             // optional — sets ai_logs.projectId
  taskId: 'uuid',                // optional — sets ai_logs.taskId
  userId: 'clerk_user_id',
  tenantId: 'org_uuid',
  operation: 'chat',             // label for filtering in ai-logs page
});
```

From a Server Action or RSC, prefer `sendAIMessage()` in `src/lib/actions/ai.ts` which handles auth automatically.

### Cost Model (in portkey.ts `calculateCost()`)

| Model prefix | Cost per 1K tokens |
|---|---|
| `gpt-4o` | $0.005 |
| `gpt-4o-mini` | $0.000150 |
| `claude-3-5-sonnet` | $0.003 |
| `claude-3-haiku` | $0.00025 |

---

## COMPLETE TEST SUITE

**Run all tests:**
```bash
npm test                  # unit + integration (Vitest, ~65 tests)
npm run test:coverage     # with lcov coverage report
npm run test:e2e          # Playwright E2E (requires: npm run dev)
npm run test:e2e:ui       # Playwright with UI mode
npx tsc --noEmit          # TypeScript type check
npm run lint              # ESLint
npm run build             # Production build
```

### Unit Tests — `src/test/unit/`

#### `ai/portkey.test.ts` (8 tests)
| Test | What it verifies |
|------|-----------------|
| calculateCost returns correct USD for gpt-4o | Cost formula: input+output tokens × rate |
| calculateCost returns correct USD for gpt-4o-mini | Cheaper rate applied |
| calculateCost returns 0 for unknown model | Graceful fallback |
| getProviderFromModel maps gpt prefix | Returns 'openai' |
| getProviderFromModel maps claude prefix | Returns 'anthropic' |
| chatCompletion saves prompt text to DB | `dal.aiLogs.create` called with `prompt: JSON.stringify(messages)` |
| chatCompletion saves completion text to DB | `dal.aiLogs.create` called with `completion: response.choices[0].message.content` |
| chatCompletion throws when PORTKEY_API_KEY missing | Environment guard |

#### `actions/projects.test.ts` (9 tests)
| Test | What it verifies |
|------|-----------------|
| createProject rejects unauthenticated user | auth() returns null userId |
| createProject rejects empty name | Zod min(1) validation |
| createProject rejects name > 255 chars | Zod max(255) validation |
| createProject sets correct tenantId | Uses org from Clerk |
| createProject returns created project | dal.projects.create return value |
| archiveProject rejects unauthenticated | auth() guard |
| archiveProject validates tenant ownership | Cannot archive another tenant's project |
| archiveProject returns updated project | Correct return value |
| createProject rejects if org not found | 404-like error when dal returns undefined |

#### `actions/ai.test.ts` (7 tests)
| Test | What it verifies |
|------|-----------------|
| sendAIMessage rejects empty message | Zod min(1) |
| sendAIMessage rejects unauthenticated user | auth() guard |
| sendAIMessage rejects if org not found | Organization lookup guard |
| sendAIMessage returns completion text | Happy path string return |
| sendAIMessage passes projectId when provided | UUID forwarded to chatCompletion metadata |
| sendAIMessage passes custom systemPrompt | System message injected |
| sendAIMessage uses default model gpt-4o-mini | No model arg → default applied |

#### `actions/tasks.test.ts` (6 tests)
| Test | What it verifies |
|------|-----------------|
| createTask requires title | Zod min(1) |
| createTask rejects unauthenticated | auth() guard |
| createTask creates with correct tenantId | Tenant isolation |
| moveTask updates columnId and status | Status derived from column |
| moveTask validates tenant ownership | Cannot move another tenant's task |
| deleteTask validates tenant ownership | Ownership check before delete |

---

### Integration Tests — `src/test/integration/`

#### `api/execute.test.ts` (7 tests)
| Test | What it verifies |
|------|-----------------|
| returns 401 when unauthenticated | Auth guard on POST |
| returns 404 when organization not found | Org lookup guard |
| returns 404 when project not found | Project lookup + tenant check |
| returns 500 when E2B_API_KEY missing | Env var guard (checked before sandbox creation) |
| returns 500 for invalid language | Zod enum, caught in try-catch |
| executes python code and returns output | Mocked Sandbox.runCode stdout → `body.output` |
| logs execution to ai_logs table | `dal.aiLogs.create` called with `operation: 'code_execution'` |

**Mocks used:** `@e2b/code-interpreter` (Sandbox.create + runCode + kill), `@clerk/nextjs/server`, `@/lib/db/dal`, `@/lib/db/index`

#### `api/analytics.test.ts` (3 tests)
| Test | What it verifies |
|------|-----------------|
| returns 401 when unauthenticated | Auth guard |
| returns 404 when organization not found | Org not found case |
| returns cost history data successfully | 200 with array body |

**Mocks used:** `@clerk/nextjs/server`, `@/lib/db/dal`, `@/lib/db/index`, `next/headers`

#### `webhooks/clerk.test.ts` (2 tests)
| Test | What it verifies |
|------|-----------------|
| processes organization.created and does not throw | Returns a Response (any valid status) |
| rejects request with missing Svix headers | Returns 4xx status |

**Mocks used:** `svix` (Webhook class), `@/lib/db/dal`, `@/lib/db/index`, `next/headers`

---

### Component Tests — `src/test/components/`

#### `kanban/task-card.test.tsx` (9 tests)
| Test | What it verifies |
|------|-----------------|
| renders task title | Title text visible |
| renders task description when provided | Description visible |
| does not render description when absent | null description = no element |
| shows the priority badge | "high"/"medium"/"low" label |
| shows AI context indicator when aiContext populated | "AI context attached" text + complexity badge |
| does not show AI context indicator when aiContext is empty | `{}` → no badge |
| shows Wrike badge when wrikeTaskId is set | "Wrike" text visible |
| does not show Wrike badge when absent | No "Wrike" text |
| shows due date when provided | Formatted "Mar 15" |
| applies dragging opacity when isDragging=true | `style.opacity === '0.4'` |

**Mocks used:** `@dnd-kit/sortable`, `@dnd-kit/utilities`

#### `analytics/ai-logs-table.test.tsx` (11 tests)
| Test | What it verifies |
|------|-----------------|
| shows empty state when no logs exist | "No AI interactions yet" |
| renders a row for each log entry | One row per log |
| truncates long prompts in table view | Displayed text < full text length |
| renders operation badge | "chat" badge visible |
| renders cost formatted to 6 decimal places | "$0.000645" text |
| shows detail dialog when a row is clicked | Radix Dialog opens via `document.body.toHaveTextContent` |
| shows full prompt text in dialog | Full content visible in dialog |
| shows full completion text in dialog | Completion visible in dialog |
| shows token breakdown in dialog | Prompt/Completion/Total Tokens labels |
| shows trace ID in dialog when present | `trace_abc123` displayed |
| shows dash when prompt is null | `—` rendered for null fields |

**Pattern note:** Dialog testing uses `document.body.toHaveTextContent()` (not `screen.getByText`) because Radix UI renders dialogs in a portal outside the React root.

---

### E2E Tests — `src/test/e2e/` (Playwright)

> Require `npm run dev` running on port 3000 with a seeded database.

#### `auth.spec.ts`
- Landing page accessible without authentication
- Sign-in page renders correctly
- Unauthenticated `/dashboard` redirects to `/sign-in`
- Post-sign-in redirects to `/dashboard`

#### `dashboard.spec.ts`
- Dashboard loads with correct heading
- Navigation items visible (Projects, Tasks, AI Logs, Analytics)
- Stats cards render (even if values are 0)
- AI Logs nav item links to correct route

#### `projects.spec.ts`
- Projects page loads
- "New Project" button is visible
- Create project modal opens on click
- Form validation prevents empty name submission

---

## ENVIRONMENT VARIABLES

```bash
# Database (required for any DB operation)
DATABASE_URL=postgresql://USER:PASSWORD@HOST/DATABASE?sslmode=require
DIRECT_URL=postgresql://USER:PASSWORD@HOST/DATABASE?sslmode=require

# Clerk Authentication (required for all auth)
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_YOUR_KEY_HERE
CLERK_SECRET_KEY=sk_test_YOUR_SECRET_HERE
CLERK_WEBHOOK_SECRET=whsec_YOUR_WEBHOOK_SECRET_HERE

# E2B Sandbox (required for /api/execute)
E2B_API_KEY=e2b_YOUR_API_KEY_HERE

# Portkey AI Gateway (required for all AI features)
PORTKEY_API_KEY=YOUR_PORTKEY_API_KEY_HERE
PORTKEY_VIRTUAL_KEY=YOUR_PORTKEY_VIRTUAL_KEY_HERE

# GitHub App (required for repo integration)
GITHUB_APP_ID=YOUR_APP_ID
GITHUB_APP_PRIVATE_KEY=-----BEGIN RSA PRIVATE KEY-----\nYOUR_KEY\n-----END RSA PRIVATE KEY-----

# Vercel API (required for deployments)
VERCEL_API_TOKEN=YOUR_VERCEL_TOKEN_HERE
VERCEL_TEAM_ID=YOUR_TEAM_ID_HERE

# Wrike (required for task sync)
WRIKE_API_TOKEN=YOUR_WRIKE_TOKEN_HERE
WRIKE_WEBHOOK_SECRET=YOUR_WRIKE_SECRET_HERE

# Upstash Redis (required for BullMQ)
UPSTASH_REDIS_URL=https://YOUR_UPSTASH_URL_HERE
UPSTASH_REDIS_TOKEN=YOUR_UPSTASH_TOKEN_HERE
```

---

## SECURITY CONSTRAINTS

### Multi-Tenancy Enforcement
- Every DB table has `tenant_id` referencing `organizations.id`
- Server Actions MUST call `auth()` and verify `orgId` before any mutation
- DAL methods accept `tenantId` and include it in every `WHERE` clause
- API routes MUST check organization membership via `dal.organizations.getByClerkOrgId(orgId)`

### Security Headers (next.config.ts)
```
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

### Secrets Policy
- Never commit real secrets — `.env.example` contains placeholders only
- Vercel environment variables use `type: "encrypted"` for injected secrets
- `DATABASE_URL` is checked at module load in `src/lib/db/index.ts` — always fails fast if missing

---

## DEVELOPMENT GUIDELINES

### Server Components First
```typescript
// ✅ Default: RSC — no useState, no useEffect
export default async function ProjectsPage() {
  const { orgId } = await auth();
  const projects = await dal.projects.getAllByTenant(orgId);
  return <ProjectList projects={projects} />;
}

// Use Client Components ONLY for: interactivity, browser APIs, event handlers
'use client'
export function ProjectList({ projects }: { projects: Project[] }) { ... }
```

### Server Actions Pattern
```typescript
'use server'
import { auth } from '@clerk/nextjs/server';
import { z } from 'zod';
import { dal } from '@/lib/db/dal';

const schema = z.object({ name: z.string().min(1).max(255) });

export async function createProject(input: unknown) {
  const { userId, orgId } = await auth();
  if (!userId || !orgId) throw new Error('Unauthorized');

  const data = schema.parse(input);  // throws ZodError on invalid input
  const org = await dal.organizations.getByClerkOrgId(orgId);
  if (!org) throw new Error('Organization not found');

  return dal.projects.create({ ...data, tenantId: org.id });
}
```

### AI Interaction Pattern
```typescript
// From a Server Action or RSC:
import { sendAIMessage } from '@/lib/actions/ai';

const result = await sendAIMessage({
  message: userInput,
  projectId: project.id,  // optional — logs against project
  systemPrompt: 'You are a code assistant.',
});
// result.completion contains the AI response
```

---

## CI/CD PIPELINE

### `.github/workflows/ci.yml`
Runs on: push to `main` or `claude/**`, PR to `main`

| Job | Steps |
|-----|-------|
| `lint-and-type-check` | `npm ci` → `npm run lint` → `npx tsc --noEmit` |
| `unit-tests` | `npm ci` → `npm run test:coverage` → Codecov upload |
| `build` | `npm ci` → `npm run build` (uses GitHub Secrets for env vars) |

### `.github/workflows/e2e.yml`
Runs on: push to `main` or manual trigger (`workflow_dispatch`)

Steps: install Playwright browsers → start dev server → run `playwright test` → upload report artifact

---

## PILOT READINESS CHECKLIST

### Critical Path (must be done before pilot)
- [x] Agent prompts saved to DB and visible in AI Logs page
- [x] Dashboard shows real AI cost and request metrics
- [x] Security headers on all routes
- [x] Secrets removed from `.env.example`
- [x] E2B code execution working
- [x] Error boundaries (no white screen crashes)
- [x] 65 unit/integration tests passing
- [x] CI/CD pipeline (auto-tests on every push)

### Nice to Have (post-pilot)
- [ ] E2E test suite passing (needs seeded DB)
- [ ] Coverage above 80%
- [ ] pgvector RAG implementation
- [ ] Real-time Kanban updates
- [ ] BullMQ media generation worker

---

## DIAGNOSING THE AGENT PROMPTS BUG (for reference)

**Root cause:** `logAIUsage()` in `src/lib/ai/portkey.ts` was calling `dal.aiLogs.create()` but omitting the `prompt` and `completion` fields — even though those columns exist in the `ai_logs` schema.

**Fix applied:**
1. Added `request: ChatCompletionRequest` parameter to `logAIUsage()`
2. Extracts `prompt = JSON.stringify(request.messages)`
3. Extracts `completion = response.choices[0]?.message?.content ?? ''`
4. Both passed to `dal.aiLogs.create({ ..., prompt, completion })`
5. `chatCompletion()` now passes `request` to `logAIUsage()`

**Verification:**
1. Set valid `PORTKEY_API_KEY` and `PORTKEY_VIRTUAL_KEY` in `.env.local`
2. Send a message via the AI chat interface
3. Navigate to `/dashboard/ai-logs` — entry must appear with full prompt + completion text
4. Navigate to `/dashboard` — AI Requests counter must show non-zero, Total Cost must show > $0.00

---

*Last Updated: 2026-02-27*
*Phase: Production Readiness Sprint (65/65 tests passing)*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cadieuxj)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cadieuxj)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
