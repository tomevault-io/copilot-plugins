## flow-desk

> This repository is designed for long-running coding-agent work. The goal is not to maximize raw code output. The goal is to leave the repo in a state where the next session can continue without guessing.

# FlowDesk — Agent Instructions

This repository is designed for long-running coding-agent work. The goal is not to maximize raw code output. The goal is to leave the repo in a state where the next session can continue without guessing.

## Tech Stack

| Layer      | Technology                                                 |
| ---------- | ---------------------------------------------------------- |
| Frontend   | React 18 + Vite + TypeScript + Tailwind CSS v4 + shadcn/ui |
| Backend    | Hono + Node.js + TypeScript                                |
| Database   | PostgreSQL 16                                              |
| Cache      | Redis 7                                                    |
| ORM        | Prisma                                                     |
| Realtime   | Socket.IO (Redis adapter)                                  |
| Auth       | JWT in httpOnly cookie (bcrypt + Google OAuth)             |
| AI         | OpenAI-compatible (custom baseUrl + model)                 |
| Deployment | Docker Compose (self-hosted)                               |

## Startup Workflow

Before writing code:

1. Confirm the working directory with `pwd` (expect `/home/thanh/flow-desk`).
2. Read `claude-progress.md` for the latest verified state and next step.
3. Read `feature_list.json` and choose the highest-priority unfinished feature.
4. Review recent commits with `git log --oneline -5`.
5. Run `./init.sh`.
6. Run the required smoke or end-to-end verification before starting new work.

If baseline verification is already failing, fix that first. Do not stack new feature work on top of a broken starting state.

## Feature workflow (`plan-feature`)

When planning or shipping a **product feature** (new capability, ROADMAP item, or work that updates `feature_list.json`):

1. **Invoke** the `plan-feature` skill before writing feature code.
2. Canonical path: `.agents/skills/plan-feature/SKILL.md` (Agent Skills standard).
3. Slash: `/plan-feature <request | ROADMAP id | next>` (Claude, Pi, OpenCode, Cursor, Grok).

The skill **orchestrates** Superpowers (`brainstorming` → `writing-plans` → `subagent-driven-development` or `executing-plans` → `verification-before-completion` → `finishing-a-development-branch`) plus FlowDesk harness rules (single active feature, schema hygiene, `pnpm verify` evidence). Do not re-implement that pipeline ad hoc in chat.

**Multi-agent (one pattern for all):** every host skill dir is a **directory symlink** to canonical trees under `.agents/skills/`; every slash command is a **file symlink** to `.pi/prompts/*.md`. Rebuild after clone: `pnpm sync:agents` (also run by `./init.sh`). Details: `.agents/skills/plan-feature/references/adapters.md`.

## Harness: FlowDesk agent team

Based on [revfactory/harness](https://github.com/revfactory/harness) (L3 team-architecture factory). Domain agents + skills for multi-role ship/review — **complements** `plan-feature`, does not replace it.

**Goal:** Coordinate explorer → implementer → security+QA fan-out → durable docs with `_workspace/` artifacts.

**Trigger:** multi-role ship/review, "flowdesk team", "ship with harness", `/flowdesk-team` → use skill `flowdesk-team`. Product feature design/plan gates → still `plan-feature`. Reconfigure agents/skills → skill `harness`.

| Kind          | Path                                                            |
| ------------- | --------------------------------------------------------------- |
| Agents        | `.claude/agents/fd-*.md`                                        |
| Orchestrator  | `.agents/skills/flowdesk-team/`                                 |
| Domain skills | `flowdesk-implement`, `flowdesk-security-review`, `flowdesk-qa` |
| Meta-factory  | `.agents/skills/harness/` (upstream workflow + references)      |

**Change log:**

| Date       | Change                                                                                | Target                                                     | Reason                                                                                                                                              |
| ---------- | ------------------------------------------------------------------------------------- | ---------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| 2026-07-18 | Initial FlowDesk team (5 agents + 4 domain skills + harness meta)                     | `.claude/agents/*`, `.agents/skills/flowdesk-*`, `harness` | Setup from revfactory/harness                                                                                                                       |
| 2026-07-18 | plan-feature v2.2: add missing Superpowers skills to inheritance + execution dispatch | `.claude/skills/plan-feature/SKILL.md`                     | Close orchestration gaps: `dispatching-parallel-agents`, `receiving-code-review`, `using-git-worktrees` now properly wired with activation triggers |
| 2026-07-18 | Phase 6 harness test PASS; harness description exclusions for product bugs/IDOR       | `harness` skill description; `_workspace/harness-test/`    | Iteration-1 validation (structure/dry-run/skill smoke/triggers)                                                                                     |

## Working Rules

- Work on **one feature at a time** (only one `in_progress` in `feature_list.json`).
- Do **not** mark a feature complete just because code was added.
- Keep changes within the selected feature scope unless a blocker forces a narrow supporting fix.
- Do **not** silently change verification rules during implementation.
- Prefer durable repo artifacts over chat summaries.

## Pre-Commit Gate (Non-Negotiable)

**Every commit must pass the full gate. No exceptions. No `--no-verify`.**

Before staging or committing, run:

```bash
pnpm verify
```

This executes both pre-commit and pre-push lefthook hooks and ensures:

| Check             | Command                              | What it catches                                      |
| ----------------- | ------------------------------------ | ---------------------------------------------------- |
| Secret scan       | `bash .githooks/pre-commit`          | Accidental credential commits                        |
| Format            | `pnpm format:check`                  | Prettier style drift                                 |
| Lint              | `pnpm lint`                          | Code quality / unused imports                        |
| Typecheck         | `pnpm typecheck`                     | Type errors across all packages                      |
| Unit tests        | `pnpm --filter ... test:unit`        | Logic regressions                                    |
| Integration tests | `pnpm --filter ... test:integration` | API contract / DB regressions                        |
| Build             | `pnpm build`                         | Compilation / bundling failures                      |
| Docs alignment    | Manual review                        | `feature_list.json`, `claude-progress.md` consistent |

If any check fails, **fix the failure first** — do not commit partial work on top of a broken state.

Single-package quick-check (before full `pnpm verify`):

```bash
pnpm --filter @flow-desk/api lint && pnpm --filter @flow-desk/api typecheck
pnpm --filter @flow-desk/web lint && pnpm --filter @flow-desk/web typecheck
pnpm --filter @flow-desk/shared lint && pnpm --filter @flow-desk/shared typecheck
```

Formatting fix shortcut:

```bash
pnpm format          # writes fixes
pnpm format:check    # verifies (CI uses this)
```

**Rule of thumb**: if `pnpm verify` would pass locally, CI will pass. If you skip it, CI will catch it — and the next agent wastes time fixing it.

## Guardrails

Automated safety checks that run in CI and locally. The goal: catch bad state before it becomes a commit, PR, or deploy.

### What runs in CI (unbypassable)

The `guardrails` job in `.github/workflows/ci.yml` runs on every push/PR to `main`:

| Check                   | What it does                                             | Fail/Warn |
| ----------------------- | -------------------------------------------------------- | --------- |
| `pnpm audit`            | Catches high-severity dependency vulnerabilities         | FAIL      |
| gitignore coverage      | Ensures `.env`, `node_modules`, `dist`, etc. are ignored | FAIL      |
| `--no-verify` detection | Warns if recent commits used `--no-verify`               | WARN      |
| console.log in source   | Flags `console.log` in non-test source files             | WARN      |

### What runs locally (`pnpm guardrails`)

```bash
pnpm guardrails          # all checks
pnpm guardrails secrets  # one category
pnpm guardrails audit    # one category
```

Categories: `secrets`, `large-files`, `lockfile`, `gitignore`, `audit`, `console`

| Check            | Threshold                                         | Action                         |
| ---------------- | ------------------------------------------------- | ------------------------------ |
| Secret scan      | Any match                                         | FAIL — fix before commit       |
| Large files      | > 1MB                                             | FAIL — use Git LFS or compress |
| Lockfile sync    | `package.json` changed without `pnpm-lock.yaml`   | FAIL — run `pnpm install`      |
| Gitignore        | Missing patterns for `.env`, `node_modules`, etc. | FAIL — add pattern             |
| Dependency audit | High-severity CVE                                 | FAIL — update/patch            |
| console.log      | In non-test `.ts`/`.tsx` source                   | FAIL — use `logger`            |

### Enforcement layers

```
Layer 1: pnpm guardrails (pre-commit)     → blocks bad state locally
Layer 2: lefthook (format+lint+typecheck)  → blocks code quality drift
Layer 3: pnpm verify (pre-push)           → full gate before push
Layer 4: CI guardrails job                → unbypassable on main
Layer 5: CI quality+tests+build           → full validation
```

**No layer is bypassable.** If you use `--no-verify`, CI will catch it. If CI is bypassed, branch protection blocks merge.

### Gitignore required patterns

```gitignore
# Secrets — NEVER commit these
.env
.env.local
.env.*.local

# Build artifacts
dist
build
.turbo

# Dependencies
node_modules

# IDE/OS
.vscode
.idea
.DS_Store

# Coverage
coverage

# Logs
*.log
```

If any pattern is missing, `pnpm guardrails gitignore` will flag it and CI will fail.

## Caveman Auto-Toggle (Planning Workflow)

Caveman compression auto-toggles around Superpowers phases to save tokens without hurting clarity where it matters:

- **Brainstorming / requirements gathering**: full verbosity — clarifying questions and design tradeoffs stay readable.
- **`writing-plans` / `executing-plans` output**: caveman full compression on.
- **Human-facing output after execution** (commit messages, PR descriptions, `claude-progress.md` session records, `feature_list.json` evidence/notes): normal verbosity — these are read outside the session.

Manual `/caveman` and `/normal mode` override at any time.

## Engineering Pipeline (Non-Negotiable)

Before writing any feature code, ensure artifacts exist:

```
PRD.md          → Problem, goals, non-goals, user stories, success metrics
ADR-XXX.md      → Context, decision, rationale, alternatives rejected, consequences
TASKS.md        → Epics → Stories → Tasks (with effort estimates)
ACCEPTANCE.md   → Concrete, testable acceptance criteria per story
RISKS.md        → Risk, likelihood, impact, mitigation for each identified risk
```

If any artifact is incomplete, **stop and request it before proceeding**.

## Architecture Standards

### Backend (Hono + Node.js)

```
apps/api/src/modules/{feature}/
  {feature}.routes.ts      # HTTP + WebSocket route registration
  {feature}.service.ts     # Business logic, orchestration
  {feature}.repository.ts  # DB access (Prisma) — no business logic
  {feature}.schema.ts      # Zod schemas for I/O validation
  {feature}.types.ts       # TypeScript types/interfaces
  {feature}.test.ts        # Unit + integration tests
apps/api/src/shared/
  middleware/              # auth, rate-limit, logging, error handler
  lib/                     # prisma client, redis client, logger, llm-provider
  errors/                  # typed error classes
```

### Frontend (React + Vite)

```
apps/web/src/features/{feature}/
  components/              # Feature-specific UI
  hooks/                   # useQuery / useMutation wrappers
  api.ts                   # Type-safe API client
  types.ts
  index.ts                 # Public API of the feature
apps/web/src/components/ui/  # Shared, headless UI primitives
apps/web/src/lib/            # queryClient, auth, socket, utils
apps/web/src/pages/          # Route-level components only (thin shells)
```

### Required Patterns

- **Zod** for all input validation.
- **Centralized error handler** — no raw `try/catch` leaking to routes.
- **Structured logging** (JSON, with `requestId`, `userId`, `duration`).
- **JWT** auth as middleware, never inline in handlers.
- **Rate limiting** per route category (auth: strict, API: moderate).
- **Redis caching** with explicit TTL and invalidation strategy.
- **TanStack Query** for all server state — no manual `useEffect` fetch.
- **Optimistic updates** on mutations where UX demands it.
- **Type-safe API client** — request/response types from Zod schemas.

### Prisma Rules

- Every model: `id` (cuid), `createdAt`, `updatedAt`, `deletedAt?` (soft delete).
- `@@index` for every FK and common filter field.
- `@@unique` for business-level uniqueness.
- Explicit `@relation` names on both sides.
- Never drop columns in a single migration — deprecate first.
- Additive migrations only in production.

### Future-Sprint Schema Hygiene (don't paint into corners)

Phase 1–2 work must not bake in assumptions that block Epic/Sprint/Board models later. Checklist for every task/board query this phase:

- **No `board` in names.** Query/repo/service names use `workspace`, not `board` (`getColumnsByWorkspace`, `listTasksByWorkspace`). A future `Board` model slots in as an additional scope, not a rename.
- **Structural fields stay minimal.** Touch only `Task.columnId` and `Task.parentTaskId` for structure. No `Task.epicId`, `Task.sprintId`, `Task.boardId` until their UI ships.
- **Filter by parameter, not hardcoded scope.** `listTasks(workspaceId, filters)` not `listTasksForWorkspaceX()` baked into SQL. A future `boardId` arg extends the signature instead of forcing a rewrite.
- **Epic = `parentTaskId` reuse.** Epic→Story→Subtask is a depth/generalization of the existing self-ref, not a new model now. No separate `Epic` relation; future story UI adds a `type` discriminator.
- **Sprint + estimation deferred together.** No `Sprint` table, no `storyPoints`/`estimate` column until sprint+estimation UI ships together (one without the other feels broken).
- **Migration stays additive.** Future Board/Epic/Sprint = new tables + nullable FKs + `@@index`, never a rewrite of existing task queries.

### Socket.IO Rules

- Namespaces by domain: `/tasks`, `/notifications`, `/collab`.
- Rooms by resource ID: `workspace:{id}`, `task:{id}`.
- Auth middleware validates JWT on `connection`, disconnects on failure.
- Memory leak guard: `socket.leave()` in `disconnect`, clean up intervals.

## Required Artifacts

- `feature_list.json` — source of truth for feature state.
- `claude-progress.md` — session log + current verified status.
- `init.sh` — standard startup + verification path.
- `session-handoff.md` — compact handoff for larger sessions (see **Session handoff format** below).

## Session handoff format (`session-handoff.md`)

Agents **must** keep `session-handoff.md` in this layout. Prefer **GitHub-flavored markdown tables** (aligned columns) over free-form prose. Do **not** invent a new structure or drop durable sections.

### Required sections (in order)

1. **Title** — `# Session Handoff — FlowDesk`
2. **Header table** — field/value rows:
   - `Last session` — short goal + date
   - `Tip branch` — e.g. `main` @ origin (avoid a tip SHA that goes stale after every amend)
   - `Code ship` / `Docs ship` — notable commit hashes + one-line subjects when useful
   - `Status` — feature_list counts, plans done, blockers
3. **`## Verified state`** — table: Check | Detail (startup, gate, unit/integration counts, URLs, demo creds)
4. **`## Shipped this session`** — short intro line, then:
   - **Primary table**: `Plan | Area | What shipped` (one row per plan/feature id; full sentences or clear fragments)
   - Optional second table: `Follow-up | Detail` (review polish, harness files)
5. **`## Open / operator`** — table: `Kind | Item` (deploy secrets, product direction)
6. **`## Commands`** — table: `Kind | Commands` with the **full durable ops set** (do not shrink this to 2–3 lines):
   - Stack: `pnpm stack:up` / `stack:up-build` / `stack:down` / `stack:logs` / `stack:ps`
   - Prisma: `pnpm db:push` / `db:migrate` / `db:seed` / `db:studio` / `db:reset`
   - Build/check: `pnpm build` / `pnpm typecheck` / `pnpm test`
   - Local dev: `pnpm dev` / `pnpm dev:reset` / `pnpm dev:turbo`
   - Hooks/gate: `pnpm setup:lefthook` / `pnpm check:secrets` / `pnpm verify`
   - Plans: `plans/README.md`

### Rules

- **Update on ship**, not only at session end, when handoff would otherwise lie about status or shipped work.
- Keep **Commands** complete — it is an ops cheat sheet, not session-specific notes.
- Run Prettier on the file so table pipes stay aligned (`pnpm exec prettier --write session-handoff.md`).
- Mirror product/security ship state into `feature_list.json`, `claude-progress.md`, and `RISKS.md` as needed; handoff is the **compact index**, not the only record.

### Skeleton (copy when rewriting)

```markdown
# Session Handoff — FlowDesk

| Field        | Value             |
| ------------ | ----------------- |
| Last session | …                 |
| Tip branch   | main @ origin (…) |
| Code ship    | `abc1234` …       |
| Docs ship    | `def5678` …       |
| Status       | …                 |

## Verified state

| Check       | Detail                            |
| ----------- | --------------------------------- |
| Startup     | …                                 |
| Gate        | …                                 |
| API unit    | **N**                             |
| Integration | **N**                             |
| Web unit    | **N**                             |
| Shared unit | **N**                             |
| Web         | http://localhost:5173             |
| API         | http://localhost:3000             |
| Demo        | `demo@flow-desk.app` / `demo1234` |

## Shipped this session

Intro line (scope + optional primary commit).

| Plan   | Area     | What shipped |
| ------ | -------- | ------------ |
| ID-001 | Security | …            |

| Follow-up | Detail |
| --------- | ------ |
| …         | …      |

## Open / operator

| Kind      | Item |
| --------- | ---- |
| Deploy    | …    |
| Direction | …    |

## Commands

| Kind          | Commands                                                                      |
| ------------- | ----------------------------------------------------------------------------- |
| Stack         | `pnpm stack:up` / `stack:up-build` / `stack:down` / `stack:logs` / `stack:ps` |
| Prisma        | `pnpm db:push` / `db:migrate` / `db:seed` / `db:studio` / `db:reset`          |
| Build / check | `pnpm build` / `pnpm typecheck` / `pnpm test`                                 |
| Local dev     | `pnpm dev` / `pnpm dev:reset` / `pnpm dev:turbo`                              |
| Hooks / gate  | `pnpm setup:lefthook` / `pnpm check:secrets` / `pnpm verify`                  |
| Plans         | `plans/README.md`                                                             |
```

Live example: current `session-handoff.md` on `main`.

## Secrets Policy

- **Never paste API keys, tokens, or credentials into chat, commit messages, or PR descriptions.** Treat them as toxic to conversation history.
- All secrets live in `.env` (gitignored) or `.env.local` (gitignored). The `LLM_API_KEY`, `JWT_SECRET`, `GOOGLE_CLIENT_SECRET`, `AWS_*` values must never appear in any committed file.
- A pre-commit hook at `.githooks/pre-commit` enforces this:
  - Blocks staged files matching `.env*` and common credential paths (PEM, `id_rsa`, `service-account*.json`, etc).
  - Greps staged content for high-confidence secret patterns (`sk-…`, `sk-ant-…`, `AIza…`, `ghp_…`, `AKIA…`, JWT, private-key blocks, `LLM_API_KEY=…`).
- Hooks are installed automatically by `./init.sh` (calls `pnpm setup:lefthook`). Configuration in `lefthook.yml`. To install manually: `pnpm setup:lefthook`. To re-run the secret check without committing: `pnpm check:secrets`. To run all gates locally: `pnpm verify`.
- **pre-commit**: secret scan (`.githooks/pre-commit`) + per-package typecheck (web/api/shared) — runs in ~15s.
- **pre-push**: full typecheck + BE integration tests + web build — runs in ~60-90s.
- If a real key is ever exposed, **rotate it at the provider immediately**. The key is compromised the moment it appears in a chat or terminal scrollback.

## Definition of Done

A feature is done **only when all** are true:

- the target behavior is implemented.
- the required verification actually ran.
- evidence is recorded in `feature_list.json` or `claude-progress.md`.
- the repository remains restartable from `./init.sh`.

## End of Session

Before ending:

1. Update `claude-progress.md` with session record.
2. Update `feature_list.json` (status, evidence).
3. Record unresolved risk/blocker (and `RISKS.md` when risks change).
4. Rewrite/update `session-handoff.md` using the **Session handoff format** above (tables; full Commands).
5. Commit with descriptive message once work is in safe state (`pnpm verify` / hooks pass).
6. Leave repo clean enough for next session to run `./init.sh` immediately.

## Anti-Patterns (Never)

- ❌ Jump to code without artifacts.
- ❌ Use `any` in TypeScript.
- ❌ Mix business logic into routes or repositories.
- ❌ Leave error paths unhandled.
- ❌ Deploy breaking DB changes in a single migration.
- ❌ Poll where WebSocket/SSE works.
- ❌ Store secrets in env vars accessible to frontend bundle.
- ❌ Praise approaches that create debt. Challenge them.
- ❌ Commit without running `pnpm verify` first.

## Golden Rule

> Simple, boring, and correct beats clever every time.
> If an architecture decision needs a long explanation to justify it, it's probably wrong.

---
> Source: [dt418/flow-desk](https://github.com/dt418/flow-desk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
