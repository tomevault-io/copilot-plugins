## orchestralay

> This is the authoritative architecture document for any AI agent or developer working in this codebase. Read this before reading source files.

# AGENTS.md — OrchestraLay architecture reference

This is the authoritative architecture document for any AI agent or developer working in this codebase. Read this before reading source files.

---

## What OrchestraLay does

OrchestraLay accepts AI task requests (code generation, debugging, refactoring, analysis, review) from developers via a CLI or REST API. It routes each task to the best available model using a 6-gate decision engine considering cost, health, and task type. It executes the task with automatic failover if a model fails. It runs every proposed code change through a safety layer that computes unified diffs and checks for dangerous operations. It presents results to developers for explicit approval before anything touches their filesystem. Every model call is logged with exact token counts and costs.

The product targets developers on Replit, Vercel, and similar platforms who suffer from two problems: unpredictable AI agent costs and agents that make breaking changes. OrchestraLay solves both.

---

## File responsibility map

Every file in the project has one job. This is the complete map.

```
server/
├── index.ts
│   Wires Express + tRPC + CORS. Starts queue then worker then server.
│   Startup order is load-bearing — see CLAUDE.md Bug 3.
│
├── db/
│   ├── schema.ts       14 Drizzle tables. Every other file imports from here.
│   └── index.ts        Drizzle client. DATABASE_URL only.
│
├── trpc/
│   ├── trpc.ts         tRPC instance, error formatter (no stack in production).
│   ├── context.ts      Resolves every request to AuthContext union type.
│   │                   Two paths: JWT (eyJ prefix) → DashboardAuth
│   │                              API key (olay_ prefix) → ApiKeyAuth
│   └── guards.ts       Reusable middleware. Exports procedure variants:
│                       publicProcedure, authedProcedure, dashboardProcedure,
│                       adminProcedure, apiKeyProcedure(scope).
│
├── routers/
│   ├── index.ts        Merges all routers into appRouter.
│   ├── auth.ts         API key CRUD (create, list, revoke). dashboardProcedure.
│   ├── tasks.ts        Task lifecycle. submit=apiKeyProcedure. Others=authed.
│   ├── diffs.ts        Diff review workflow. 8 procedures.
│   └── dashboard.ts    Aggregated metrics. getOverview + getCosts. dashboardProcedure only.
│
├── workers/
│   └── orchestrateTask.ts
│       pg-boss consumer. Owns the complete task lifecycle.
│       Coordinates: router → model caller → diff engine → cost logger → broadcast.
│       teamSize: 5, teamConcurrency: 3.
│
└── lib/
    ├── supabase.ts         Admin client (server only) + anon client (JWT validation).
    ├── hashKey.ts          generateApiKey() → olay_ prefix + 32 random bytes.
    │                       hashApiKey() → SHA-256. Never bcrypt.
    ├── tokenizer.ts        estimateTokens(text) → Math.ceil(text.length / 4).
    │                       Called before resolveModel(). Must exist or worker crashes.
    ├── queue.ts            pg-boss singleton. getQueue() is the only export.
    ├── modelRegistry.ts    6 model specs with pricing, avgOutputTokens per task type,
    │                       strengths[], maxConcurrentRequests.
    │                       estimateCostCents(modelId, taskType, promptTokens).
    │                       DEFAULT_MODEL_RANKING per task type.
    ├── modelHealth.ts      In-memory circuit breaker. State lives in module scope.
    │                       recordSuccess() / recordFailure() / isModelAvailable().
    │                       threshold=3 failures, cooldown=60s. Resets on restart.
    ├── modelRouter.ts      resolveModel() — 6-gate decision, returns RouterDecision.
    │                       resolveFailover() — iterate fallbackChain for retry.
    │                       Pure logic — no side effects except DB read for concurrency.
    ├── modelCallers.ts     callModel() dispatches to callAnthropic / callOpenAI /
    │                       callPerplexity. All accept AbortSignal. Return typed result
    │                       with real token counts and calculated cost in cents.
    ├── outputParser.ts     Parses <file_changes> XML from model output.
    │                       sanitizePath() strips ../, leading /, backslashes.
    │                       Returns ParsedFileOperation[].
    ├── diffComputer.ts     computeDiff(before, after, operation) using 'diff' package.
    │                       Returns hunks, line counts, binary detection.
    ├── safetyRules.ts      checkSafetyRules(op, projectRules, overrides).
    │                       8 checks. Returns SafetyViolation[] with severity.
    ├── diffEngine.ts       Orchestrates: parse → diff → safety → persist → broadcast.
    │                       Never writes files to disk. Produces data only.
    ├── budgetGuard.ts      enforceBudget() — checks team monthly cap + project cap.
    │                       Throws FORBIDDEN with human-readable message if exceeded.
    ├── rateLimiter.ts      enforceRateLimit(keyId) — per_minute + per_day buckets.
    │                       Postgres-backed. Throws TOO_MANY_REQUESTS with retry-after.
    ├── realtime.ts         broadcastTaskUpdate(taskId, payload) via Supabase channel.
    │                       Called by worker at each status transition.
    ├── eventEmitter.ts     emitEvent(event, payload) — fire-and-forget POST to n8n.
    │                       3s AbortSignal timeout. Silently skips if N8N_WEBHOOK_URL unset.
    └── audit.ts            writeAuditLog(entry) — insert to audit_logs. Always fire-and-forget.

src/                        Vite React 19 frontend. TailwindCSS. Wouter routing.
├── App.tsx                 Three routes: / → Overview, /costs → Costs, /diffs → DiffReview.
├── lib/
│   ├── trpc.ts             tRPC client pointing at /trpc.
│   └── supabase.ts         Supabase anon client for Realtime subscription.
└── pages/
    ├── Overview.tsx        Live task feed + 4 metric cards. Realtime via Supabase.
    ├── Costs.tsx           7-day stacked bar chart + model breakdown table.
    └── DiffReview.tsx      Pending diffs across team. Approve / reject per diff.

cli/
└── index.ts                Three commands: submit, status, apply.
                            submit: POST tasks.submit → poll every 2s → print result.
                            status: GET tasks.getStatus → print formatted output.
                            apply: fetch approved diffs → write files → markApplied.
                            Reads ORCHESTRALAY_API_KEY from env.
```

---

## Data flow — task submission to file change on disk

```
Developer CLI/SDK
  │
  POST /trpc/tasks.submit (Authorization: Bearer olay_xxx)
  │
  ├─ resolveAuth() → hash token → lookup api_keys → ApiKeyAuth context
  ├─ enforceRateLimit(keyId)             ← throws if over per_minute or per_day
  ├─ enforceBudget(projectId, teamId)    ← throws if team monthly cap exceeded
  ├─ INSERT tasks (status='submitted')
  ├─ queue.send('orchestrate-task', payload)   ← fire and forget
  └─ return { taskId, realtimeChannel: 'task:<id>' }

Developer subscribes to Supabase Realtime channel 'task:<id>'

pg-boss picks up job (teamSize:5, teamConcurrency:3)
  │
  ├─ UPDATE tasks status='routing'
  ├─ broadcastTaskUpdate → Realtime → dashboard updates
  ├─ estimateTokens(prompt)
  ├─ resolveModel(input)          ← 6-gate decision (see below)
  │     reasoning[] stored in tasks.metadata
  ├─ UPDATE tasks status='executing', store routing decision
  ├─ broadcastTaskUpdate with modelName
  │
  ├─ [retry loop]
  │   callModel({ model, prompt, taskType, abortSignal: AbortSignal.timeout(ms) })
  │   │
  │   ├─ success path:
  │   │   recordSuccess(model, latencyMs)
  │   │   INSERT model_results (tokens, cost, content)
  │   │   INSERT cost_logs
  │   │   UPDATE teams SET current_month_spend_cents = current_month_spend_cents + X
  │   │   break loop
  │   │
  │   └─ failure path:
  │       recordFailure(model)             ← may open circuit breaker
  │       INSERT failed model_result
  │       broadcastTaskUpdate 'trying next model...'
  │       resolveFailover() → next candidate or null
  │       if null: UPDATE tasks status='failed', broadcast, return
  │
  ├─ runDiffEngine(taskId, modelResultId, content, projectId)
  │   ├─ parseModelOutput(content)         ← extract <file_changes> XML
  │   ├─ for each ParsedFileOperation:
  │   │   computeDiff(before, after)
  │   │   checkSafetyRules(op, projectRules, safetyOverrides)
  │   │   INSERT diffs (flagged, blocked, safetyViolations, hunks)
  │   └─ broadcastTaskUpdate with diff summary
  │
  ├─ UPDATE tasks status='completed'
  ├─ broadcastTaskUpdate status='completed', costCents
  └─ emitEvent('task.completed', payload)  ← fire and forget to n8n

Developer sees diff preview in dashboard or via CLI status
  │
  diffs.approve(diffId)      ← blocked=true cannot be approved here
  │
  CLI: orchestralay apply --task-id <id>
  │
  ├─ diffs.getForTask(taskId)         ← fetch approved, non-applied diffs
  ├─ diffs.getContent(diffId)         ← fetch beforeContent + afterContent
  ├─ fs.promises.writeFile / unlink   ← writes to developer's filesystem
  └─ diffs.markApplied(diffIds[])     ← mark applied in DB
```

---

## Auth context — union type

```typescript
type AuthContext =
  | { type: 'dashboard'; userId: string; teamId: string; role: string }
  | { type: 'apikey';   projectId: string; teamId: string; scopes: string[]; keyId: string }
  | { type: 'none' }
```

`teamId` is present on both authenticated types. Every downstream cost/billing/rate-limit check uses `teamId` — never `userId` or `projectId` alone.

JWT validation path calls `supabaseAnon.auth.getUser(token)` — server-side, never decode client-side. `req` must be passed into `resolveJwt` to read `req.query.teamId` for multi-team users.

API key validation: SHA-256 hash the raw token, query `api_keys` where `key_hash = hash AND revoked = false AND (expires_at IS NULL OR expires_at > now())`. Load `project.teamId` via join.

---

## 6-gate routing decision

`resolveModel(input)` runs 6 gates in sequence. Each gate eliminates candidates; never adds.

```
Gate 1 — Preference
  if preferredModels[] provided + valid → use that order
  else → use DEFAULT_MODEL_RANKING[taskType]

Gate 2 — Budget
  for each candidate: estimateCostCents(model, taskType, promptTokens)
  remove if estimated > budgetCents
  if ALL removed → keep cheapest anyway (never fully block on budget)

Gate 3 — Health
  remove if isModelAvailable(model) === false (circuit open)
  if ALL removed → proceed with first anyway (fail gracefully, not silently)

Gate 4 — Concurrency
  query cost_logs for active requests per model in last 60s grouped by model_name
  remove if count >= spec.maxConcurrentRequests
  if ALL removed → proceed with first anyway

Gate 5 — Select
  first remaining candidate wins

Gate 6 — Return
  { selectedModel, estimatedCostCents, reasoning: string[], fallbackChain: ModelId[] }
  reasoning[] is stored in tasks.metadata — visible in dashboard
```

`resolveFailover(failedModel, fallbackChain, taskType, promptTokens, budgetCents)`: iterate `fallbackChain`, skip models that are over budget or circuit-open, return first viable or `null`.

---

## Model registry

| Model | Provider | Input ¢/1M | Output ¢/1M | Strengths |
|---|---|---|---|---|
| claude-3-5-sonnet | anthropic | 300 | 1500 | code_generation, refactoring, review |
| claude-3-haiku | anthropic | 25 | 125 | debugging, analysis |
| gpt-4o | openai | 250 | 1000 | analysis, review, debugging |
| gpt-4o-mini | openai | 15 | 60 | analysis |
| perplexity-sonar-pro | perplexity | 300 | 1500 | analysis (web-grounded) |
| perplexity-sonar | perplexity | 80 | 80 | analysis (budget) |

Costs are integer USD cents per 1M tokens. `estimateCostCents` uses pre-measured average output token counts per task type. Real cost uses actual token counts returned by the API.

Perplexity uses the OpenAI SDK pointed at `https://api.perplexity.ai` with `PERPLEXITY_API_KEY`.

---

## Database — 14 tables, relationships

```
users
  └─ team_members ──► teams
                        └─ projects
                             ├─ api_keys
                             │    └─ rate_limit_buckets
                             ├─ tasks
                             │    ├─ model_results
                             │    └─ diffs
                             ├─ integrations
                             └─ webhooks
                        ├─ cost_logs
                        └─ team_billing_history
feature_flags            (standalone, team-agnostic)
audit_logs               (nullable FKs — survives cascade deletes)
```

All FKs use `ON DELETE CASCADE` except `audit_logs` which uses `SET NULL`.
`billing_period` (format: `YYYY-MM`) is the primary dimension for all cost aggregation — always include it in GROUP BY.
`cost_cents` is always integer cents. Never decimal.

---

## Diff engine pipeline — 4 files, 1 responsibility each

```
outputParser.ts     Parse <file_changes> XML → ParsedFileOperation[]
                    sanitizePath() strips ../, leading slash, backslashes
                    Invalid paths are silently skipped

diffComputer.ts     computeDiff(before, after, operation)
                    Uses 'diff' npm package diffLines()
                    Returns { hunks[], linesAdded, linesRemoved, isBinaryFile }
                    Binary detection: null bytes in first 8000 chars

safetyRules.ts      checkSafetyRules(op, projectRules, taskOverrides)
                    Returns SafetyViolation[] — empty array = safe
                    taskOverrides override projectRules (admin scope required)

diffEngine.ts       Orchestrates the above three in sequence
                    Persists one diffs row per file operation
                    Broadcasts preview via Realtime
                    NEVER writes files to disk — that is the CLI's job
```

---

## Safety rules reference

| Rule | File patterns | Default | Severity |
|---|---|---|---|
| `protected_file` | `.env*`, `*.lock`, `*.lockb`, `package-lock.json` | always on | block |
| `file_deletion` | any file | allowFileDeletion=false | block |
| `framework_change` | `package.json`, `tsconfig.json`, `vite.config.*`, `next.config.*`, `tailwind.config.*` | allowFrameworkChanges=false | block |
| `config_file_change` | `*.config.*`, `.eslintrc`, `.prettierrc`, `Dockerfile`, `docker-compose.*` | always on | warn |
| `test_deletion` | `*.test.*`, `*.spec.*`, `__tests__/`, `/tests?/` | allowTestFileDeletion=false | block |
| `custom_blocked_path` | `project.safetyRules.customBlockedPaths[]` | empty | block |
| `large_change` | before>50 lines AND ratio>80% | always on | warn |
| `potential_secret` | regex: `api_key=`, `sk-[a-z0-9]{20,}`, JWT, `PRIVATE KEY` | always on | block |

`blocked=true` diffs cannot be approved via `diffs.approve()` — the endpoint throws FORBIDDEN. This is intentional: a human must change the project safety settings.

---

## tRPC procedure matrix

| Procedure | Guard | Caller |
|---|---|---|
| `tasks.submit` | `apiKeyProcedure('tasks:write')` | SDK / CLI |
| `tasks.getStatus` | `authedProcedure` | Both |
| `tasks.list` | `dashboardProcedure` | Dashboard |
| `tasks.cancel` | `authedProcedure` | Both |
| `diffs.getForTask` | `authedProcedure` | Both |
| `diffs.getPendingForTeam` | `dashboardProcedure` | Dashboard |
| `diffs.getContent` | `authedProcedure` | Both |
| `diffs.approve` | `authedProcedure` | Both |
| `diffs.reject` | `authedProcedure` | Both |
| `diffs.approveAll` | `authedProcedure` | Both |
| `diffs.markApplied` | `apiKeyProcedure('tasks:write')` | SDK / CLI |
| `diffs.revert` | `authedProcedure` | Both |
| `dashboard.getOverview` | `dashboardProcedure` | Dashboard |
| `dashboard.getCosts` | `dashboardProcedure` | Dashboard |
| `auth.createApiKey` | `dashboardProcedure` | Dashboard |
| `auth.listApiKeys` | `dashboardProcedure` | Dashboard |
| `auth.revokeApiKey` | `dashboardProcedure` | Dashboard |

---

## n8n integration boundary

n8n is used for outbound integrations only. It is never in the request/response critical path.

The boundary is `server/lib/eventEmitter.ts`. It fires a POST with 3s AbortSignal timeout, catches all errors, and never throws. If `N8N_WEBHOOK_URL` is not set, it silently returns.

Events and their downstream workflows:

| Event | n8n workflow responsibility |
|---|---|
| `task.completed` | Customer webhook delivery (with retry), Slack notification, Linear/GitHub comment |
| `task.failed` | Alert to project owner |
| `diff.flagged` | Safety alert to team admin |
| `cost.threshold_exceeded` | Billing alert email |

The n8n instance is at `designtec.app.n8n.cloud`. Workflows are triggered via webhook. All four workflows can be disabled without affecting core product functionality.

---

## Frontend — three pages only (MVP)

```
/ → Overview
    4 metric cards: tasks today, cost today, pending diffs, failed count
    Live task feed table: id | prompt | model | status | cost | age
    Realtime: Supabase postgres_changes on tasks table, filter team_id
    Fallback: poll dashboard.getOverview every 30s

/costs → Costs
    Month-to-date with budget progress bar (warn at 80%, danger at 100%)
    7-day stacked bar chart via Chart.js (claude=blue, gpt=teal, perplexity=amber)
    Model breakdown: model | requests | tokens | cost | %

/diffs → DiffReview
    Warning banner if any blocked diffs present
    "Approve all safe" button: approveAll({ skipFlagged: false })
    Per-diff row: operation badge | path | +N -N | model | task_id | violations
    Blocked diffs: disabled "Blocked by safety rule" button
    Realtime updates via diffs.getPendingForTeam refetch on approve/reject
```

---

## CLI commands

```bash
orchestralay submit \
  --prompt "..." \
  --type code_generation|debugging|refactoring|analysis|review \
  [--model <modelId>] \
  [--budget <cents>] \
  [--timeout <seconds>]
# Submits task, polls getStatus every 2s, prints final cost + diff count

orchestralay status --task-id <id>
# Prints status, model used, cost, pending diff count, routing reasoning

orchestralay apply --task-id <id> [--dry-run] [--revert]
# Fetches approved diffs, writes files to disk, calls markApplied
# --dry-run: print changes without writing
# --revert: call diffs.revert, restore beforeContent to disk
```

API key read from `ORCHESTRALAY_API_KEY` environment variable.

---

## Environment variables

```
SUPABASE_URL                 New project — not TTML's
SUPABASE_ANON_KEY            Public
SUPABASE_SERVICE_ROLE_KEY    Server only — never send to frontend
DATABASE_URL                 Postgres connection string from same project
ANTHROPIC_API_KEY
OPENAI_API_KEY
PERPLEXITY_API_KEY
ALLOWED_ORIGINS              Comma-separated, e.g. http://localhost:5173,https://app.orchestralay.com
N8N_WEBHOOK_URL              Optional
N8N_WEBHOOK_SECRET           Optional HMAC signing secret
STRIPE_SECRET_KEY            Week 3
STRIPE_WEBHOOK_SECRET        Week 3
PORT                         Default 3001
NODE_ENV                     development | production
```

---

## Pricing

| Plan | Price | Monthly tokens | Team seats |
|---|---|---|---|
| Starter | $29/mo | 500k | 1 |
| Pro | $99/mo | 2M | 5 |
| Enterprise | Custom | Unlimited | Unlimited |

7-day free trial via Stripe trial periods. No free tier after trial. Overage: $0.002 per 1k tokens — charge, do not hard-block.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jamilahmedansari)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jamilahmedansari)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
