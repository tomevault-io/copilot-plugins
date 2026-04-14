## real-estate-app

> > Project-specific instructions for Claude Code. Complements `memory/MEMORY.md` (stable knowledge) and `docs/pm/SPRINT_STATE.md` (sprint state).

# CLAUDE.md — Master Prompt for RealFlow

> Project-specific instructions for Claude Code. Complements `memory/MEMORY.md` (stable knowledge) and `docs/pm/SPRINT_STATE.md` (sprint state).

---

## Session Start Protocol

Every new conversation MUST begin by reading the PM brain:

1. Read `docs/pm/SPRINT_STATE.md` — current sprint, feature lifecycle, session handoff
2. Resume from "Last Session Handoff" section — do not re-discover context
3. If sprint state is stale (>24h), ask user for status update

---

## Project Identity

**Project Name:** RealFlow
**Tagline:** The frictionless CRM and workflow platform for Australian real estate agents and buyers agents.

---

## Stack Quick Reference

| Layer      | Tech                                                                                     | Dev command                             |
| ---------- | ---------------------------------------------------------------------------------------- | --------------------------------------- |
| API        | Fastify 5 / TypeScript                                                                   | `npm run dev --filter=@realflow/api`    |
| Web        | Next.js 16 (App Router) / Tailwind                                                       | `npm run dev --filter=@realflow/web`    |
| Portal     | Next.js 16 (App Router) / Tailwind                                                       | `npm run dev --filter=@realflow/portal` |
| Mobile     | React Native (Expo 54) / NativeWind                                                      | `cd apps/mobile && npx expo start`      |
| Database   | Supabase (PostgreSQL + RLS)                                                              | Supabase MCP                            |
| API deploy | Render                                                                                   | Render MCP (`srv-d6logk450q8c73a884pg`) |
| Shared     | `@realflow/shared`, `@realflow/business-logic`, `@realflow/integrations`, `@realflow/ui` | Built by Turbo                          |

---

## Architecture

- **Monorepo:** Turborepo with npm workspaces
- **Web:** Next.js 16 (App Router) with Tailwind CSS, React Query, Zustand
- **Mobile:** React Native (Expo) with NativeWind, Expo Router
- **API:** Fastify with typed Supabase client
- **Database:** PostgreSQL via Supabase with Row Level Security
- **Auth:** Supabase Auth (email/password, Google, Apple)
- **Real-time:** Supabase Realtime
- **AI:** AnthropicClient (raw fetch) in `packages/integrations/src/ai/`

---

## Development Lifecycle

Every feature follows 10 phases. Automated phases run without asking — stop only at human gates.

```
DISCOVER → PLAN → BUILD → TEST → QUALITY → HARDEN → DOCUMENT → DEPLOY → MONITOR → FINISH
   🔐        🔐      ⚡       ⚡       ⚡         ⚡          ⚡         🔐          ⚡       🔐
```

| #   | Phase    | Entry condition               | Human gate? | Auto-runs                             |
| --- | -------- | ----------------------------- | ----------- | ------------------------------------- |
| 1   | DISCOVER | New feature or sprint kickoff | Yes         | —                                     |
| 2   | PLAN     | Discovery doc signed off      | Yes         | —                                     |
| 3   | BUILD    | Contracts agreed              | No          | Code generation                       |
| 4   | TEST     | BUILD complete                | No          | `npm run test`, coverage check        |
| 5   | QUALITY  | Tests passing                 | No          | `npm run quality` (lint + type-check) |
| 6   | HARDEN   | Quality green                 | No          | Security scan + perf audit            |
| 7   | DOCUMENT | Harden complete               | No          | Doc generation                        |
| 8   | DEPLOY   | Docs complete, staging only   | Yes (prod)  | Render MCP deploy + smoke test        |
| 9   | MONITOR  | 24–48h post-deploy            | No          | Health check via Render MCP           |
| 10  | FINISH   | Monitor clean                 | Yes         | Sprint close                          |

**Automation rule:** Between human gates, Claude chains phases automatically. No "shall I continue?" — just do it. Stop only on CRITICAL failure or human gate.

---

## Automated Bash Sequences

These bash commands are pre-approved to run without confirmation during their respective phases:

### BUILD phase

```bash
npm run build                                    # Full monorepo build
npm run build -- --filter=@realflow/api          # API only
npm run build -- --filter=@realflow/web          # Web only
npm run build -- --filter=@realflow/portal       # Portal only
```

### TEST phase

```bash
npm run test                                     # Full test suite
npm run test -- --filter=@realflow/api           # API tests only
npm run test -- --filter=@realflow/business-logic # BL tests only
npm run test:coverage                            # Coverage report
npx vitest run --reporter=verbose                # Verbose output
```

### QUALITY phase

```bash
npm run quality                                  # lint + type-check (turbo task)
npm run lint                                     # ESLint all packages
npm run lint -- --fix                            # Auto-fix lint issues
npm run type-check                               # TypeScript strict
npx prettier --check .                           # Format check
npx prettier --write .                           # Auto-format
```

### DEPLOY phase

```bash
npm run build                                    # Verify build passes
# Render MCP: mcp__render__trigger_deploy         # Deploy to staging/prod
# Render MCP: mcp__render__get_deploy_logs        # Check deploy status
curl -s https://realflow-api.onrender.com/health  # Health check
curl -s https://realflow-api-staging.onrender.com/health  # Staging health
```

### GIT operations (pre-approved during BUILD)

```bash
git status                                       # Check working tree
git diff --stat                                  # Review changes
git log --oneline -10                            # Recent commits
git add <specific-files>                         # Stage changes (never git add -A)
```

---

## Command Reference

### DISCOVER

| Command                  | Agent                  | Auto-bash | Output                             |
| ------------------------ | ---------------------- | --------- | ---------------------------------- |
| `/discover $FEATURE`     | `requirements-analyst` | —         | `docs/discovery/$FEATURE.md`       |
| `/user-stories $FEATURE` | `requirements-analyst` | —         | Story map with acceptance criteria |

### PLAN

| Command                     | Agent               | Auto-bash                       | Output                                    |
| --------------------------- | ------------------- | ------------------------------- | ----------------------------------------- |
| `/sprint-plan $N: features` | `system-architect`  | `npm run test` (baseline count) | `docs/sprints/SPRINT_$N_PLAN.md`          |
| `/db-design $FEATURE`       | `backend-architect` | —                               | Migration SQL + Zod stubs + RLS checklist |
| `/api-design $FEATURE`      | `backend-architect` | —                               | `docs/api/$FEATURE.md`                    |
| `/architect $FEATURE`       | `system-architect`  | —                               | Architecture doc with ADR                 |
| `/feature-plan $FEATURE`    | `system-architect`  | —                               | Implementation plan                       |
| `/workflow-design $FEATURE` | `backend-architect` | —                               | Automation workflow design                |

### BUILD

| Command                  | Agent                | Auto-bash                                   | Output                        |
| ------------------------ | -------------------- | ------------------------------------------- | ----------------------------- |
| `/build-db $FEATURE`     | `backend-architect`  | Supabase MCP migration + `npm run db:types` | Tables + types + RLS          |
| `/build-engine $ENGINE`  | `backend-architect`  | —                                           | Engine class + test skeleton  |
| `/build-mobile $FEATURE` | `frontend-architect` | —                                           | Expo Router screens           |
| `/api-new $ROUTE`        | skill                | —                                           | Fastify route with validation |
| `/component-new $SPEC`   | skill                | —                                           | React component               |
| `/page-new $SPEC`        | skill                | —                                           | Next.js page                  |
| `/supabase:types-gen`    | skill                | `npm run db:types`                          | TypeScript types from schema  |

### TEST

| Command                      | Agent         | Auto-bash                                | Output                      |
| ---------------------------- | ------------- | ---------------------------------------- | --------------------------- |
| `/test-unit $FILE`           | `qa-engineer` | `npm run test`                           | Vitest unit tests           |
| `/test-integration $FEATURE` | `qa-engineer` | `npm run test`                           | Route → engine → DB tests   |
| `/test-coverage`             | `qa-engineer` | `npm run test:coverage`                  | Coverage report vs baseline |
| `/test-mobile $SCREEN`       | `qa-engineer` | `npm run test -- --filter=mobile`        | RN component tests          |
| `/api-test $ROUTE`           | skill         | `npm run test -- --filter=@realflow/api` | API endpoint tests          |

### QUALITY

| Command                | Agent                  | Auto-bash                                   | Output                                  |
| ---------------------- | ---------------------- | ------------------------------------------- | --------------------------------------- |
| `/quality-check`       | —                      | `npm run quality && npx prettier --check .` | All gates: lint → type-check → prettier |
| `/pr-review $BRANCH`   | `refactoring-expert`   | `git diff main...$BRANCH --stat`            | Structured code review                  |
| `/lint`                | skill                  | `npm run lint -- --fix`                     | ESLint + Prettier fix                   |
| `/code-cleanup $FILE`  | `refactoring-expert`   | —                                           | Refactor                                |
| `/code-optimize $FILE` | `performance-engineer` | —                                           | Performance optimisation                |

### HARDEN

| Command             | Agent                                        | Auto-bash                         | Output                          |
| ------------------- | -------------------------------------------- | --------------------------------- | ------------------------------- |
| `/harden $SPRINT`   | `security-engineer` + `performance-engineer` | `npm run quality && npm run test` | `docs/harden/$SPRINT.md`        |
| `/security-scan`    | `security-engineer`                          | —                                 | OWASP audit, RLS, Zod, secrets  |
| `/perf-audit`       | `performance-engineer`                       | —                                 | N+1, bundle, re-renders, <200ms |
| `/error-boundaries` | `qa-engineer`                                | —                                 | Missing error handling audit    |

### DOCUMENT

| Command                | Agent              | Auto-bash                            | Output                             |
| ---------------------- | ------------------ | ------------------------------------ | ---------------------------------- |
| `/api-docs $FEATURE`   | `technical-writer` | —                                    | `docs/api/$FEATURE.md`             |
| `/changelog $SPRINT`   | `technical-writer` | `git log --oneline` (since last tag) | Changelog entry                    |
| `/sprint-report $N`    | `technical-writer` | `npm run test` (final count)         | `docs/sprints/SPRINT_$N_REPORT.md` |
| `/docs-generate $FILE` | skill              | —                                    | JSDoc/TSDoc                        |

### DEPLOY

| Command              | Agent             | Auto-bash                                          | Output                |
| -------------------- | ----------------- | -------------------------------------------------- | --------------------- |
| `/deploy-check`      | `devops-engineer` | `npm run build && npm run test && npm run quality` | Pre-deploy checklist  |
| `/deploy-staging`    | `devops-engineer` | Render MCP deploy + `/smoke-test`                  | Staging deployment    |
| `/deploy-production` | `devops-engineer` | **Human confirmation required**                    | Production deployment |
| `/smoke-test $URL`   | —                 | `curl` commands against URL                        | 5-point verification  |

### MONITOR

| Command                | Agent               | Auto-bash                          | Output                |
| ---------------------- | ------------------- | ---------------------------------- | --------------------- |
| `/health-check`        | `devops-engineer`   | Render MCP status + `curl /health` | Service health report |
| `/error-triage $ERROR` | `backend-architect` | Render MCP logs                    | Root cause analysis   |

### SPRINT LIFECYCLE

| Command         | Agent            | Auto-bash                    | Output                           |
| --------------- | ---------------- | ---------------------------- | -------------------------------- |
| `/sprint-start` | `sprint-manager` | `npm run test` (baseline)    | Sprint plan + discovery docs     |
| `/sprint-close` | `sprint-manager` | `npm run test` (final count) | Report + MEMORY.md + tag command |

### META

| Command          | Agent      | Auto-bash                                | Output                                   |
| ---------------- | ---------- | ---------------------------------------- | ---------------------------------------- |
| `/ship $FEATURE` | chains all | All quality + harden + doc + deploy bash | "Ready for production" or failure report |

---

## The `/ship $FEATURE` Command

Chains phases 5–8 automatically. Stops on first CRITICAL failure.

```
Step 1: npm run quality              (lint + type-check — auto)
Step 2: npx prettier --check .       (format check — auto)
Step 3: npm run test                 (full suite — auto)
Step 4: npm run test:coverage        (coverage vs baseline — auto)
Step 5: /security-scan               (OWASP audit — auto)
Step 6: /perf-audit                  (N+1, bundle — auto)
Step 7: /api-docs $FEATURE           (generate docs — auto)
Step 8: /changelog                   (write changelog — auto)
Step 9: /deploy-check               (pre-deploy checklist — auto)
Step 10: /deploy-staging             (Render MCP + smoke test — auto)
```

On success: _"$FEATURE is ready for production. Run `/deploy-production` to complete."_
On failure: Stops immediately, reports which step failed and why.

**Never auto-deploys to production.** Production is always a separate, explicit `/deploy-production`.

---

## `/smoke-test $URL` — Automated Checks

Run these 5 checks against the given URL:

```bash
# 1. Health endpoint
curl -sf "$URL/health" | jq .

# 2. Auth endpoint exists (expects 400 without body, not 404)
curl -s -o /dev/null -w "%{http_code}" -X POST "$URL/api/v1/auth/login"

# 3. Protected route returns 401 without token
curl -s -o /dev/null -w "%{http_code}" "$URL/api/v1/contacts"

# 4. Invalid token returns 401
curl -s -o /dev/null -w "%{http_code}" -H "Authorization: Bearer invalid" "$URL/api/v1/contacts"

# 5. Supabase Realtime accessible (websocket upgrade check)
curl -s -o /dev/null -w "%{http_code}" "$URL/api/v1/properties"
```

Report PASS/FAIL per check. Any FAIL blocks production deploy.

---

## Agent Routing

| Task                                  | Agent                  | When         |
| ------------------------------------- | ---------------------- | ------------ |
| Ambiguous requirements → specs        | `requirements-analyst` | DISCOVER     |
| Architecture decisions, system design | `system-architect`     | PLAN         |
| DB schema, engines, route design      | `backend-architect`    | PLAN + BUILD |
| React components, state, UX           | `frontend-architect`   | PLAN + BUILD |
| Test strategy, coverage analysis      | `qa-engineer`          | TEST         |
| OWASP audit, RLS, secrets             | `security-engineer`    | HARDEN       |
| N+1, bundle, performance              | `performance-engineer` | HARDEN       |
| API docs, changelogs                  | `technical-writer`     | DOCUMENT     |
| Render deploy, CI/CD                  | `devops-engineer`      | DEPLOY       |
| Sprint lifecycle                      | `sprint-manager`       | SPRINT       |
| Code quality, tech debt               | `refactoring-expert`   | QUALITY      |
| Deep research                         | `deep-research-agent`  | DISCOVER     |
| Explain code / teach                  | `learning-guide`       | Anytime      |

---

## Session End Protocol

Before ending any session:

1. Update `docs/pm/SPRINT_STATE.md` → "Last Session Handoff" section with:
   - **Date:** today
   - **Working on:** what you were doing
   - **Status:** what's done, what's in progress
   - **Decisions:** any decisions made this session
   - **Next session:** what to do next
   - **Open questions:** anything unresolved
2. Update feature lifecycle table if any features changed phase
3. Update sprint metrics if test counts or coverage changed

---

## Coding Standards

- TypeScript strict mode — no `any` types
- Zod schemas for all API inputs and database types (defined in `packages/shared/`)
- Shared types in `packages/shared/` — never duplicate type definitions
- Components: small, composable, single-responsibility
- Custom hooks for all data fetching (prefixed with `use`)
- Optimistic updates for all mutations
- Soft deletes everywhere — never hard delete
- Conventional commits (feat:, fix:, chore:, etc.)
- Mobile-first design — every feature must work on phones

---

## Key Commands

```bash
npm run dev          # Run all apps
npm run build        # Build all
npm run lint         # Lint all
npm run test         # Test all
npm run quality      # Lint + type-check (turbo)
npm run test:coverage # Coverage report
npm run db:migrate   # Apply Supabase migrations
npm run db:reset     # Reset + seed database
npm run db:types     # Regenerate TypeScript types from Supabase
```

---

## Key Modules

1. **CRM** — Contacts with buyer/seller profiles, duplicate detection, lead scoring, activity timeline
2. **Pipeline** — Buyer (8 stages) and Seller (6 stages) pipelines with validated transitions
3. **Properties** — Listings with portal sync (Domain, REA), media, analytics
4. **Workflows** — Trigger → condition → action automation engine
5. **Integrations** — Domain.com.au, Meta (Facebook/Instagram), LinkedIn
6. **Communication Hub** — Email, SMS, phone, social DMs in one place
7. **Client Portal** — Buyer-facing dashboard with documents, progress, messaging
8. **Property Alerts** — Subscription-based alert engine for new listings
9. **Social Leads** — DM → CRM lead ingestion from Meta/Instagram
10. **Off-Market** — Private property network for buyers agents
11. **Team Management** — Round-robin assignment, performance snapshots

---

## Design Principles

1. Mobile-first — agents live on their phones
2. Speed over features — pages load < 200ms
3. Zero-friction lead capture — < 60s from enquiry to agent notification
4. Opinionated workflows — ship with best-practice templates
5. Social-native — one-tap listing posts, DM → CRM ingestion
6. Never lose data — soft deletes, full audit trail
7. Australian market first — AUD, AU addresses, AU portal integrations

---

## Quality Gates

| Gate              | Command                 | Blocks            | Runs During         |
| ----------------- | ----------------------- | ----------------- | ------------------- |
| ESLint            | `npm run lint`          | Commit, CI        | QUALITY, pre-commit |
| TypeScript strict | `npm run type-check`    | Commit, CI        | QUALITY, pre-commit |
| Vitest suite      | `npm run test`          | Commit, CI        | TEST, pre-commit    |
| Quality bundle    | `npm run quality`       | Deploy            | QUALITY             |
| Coverage          | `npm run test:coverage` | Report only       | TEST                |
| Security scan     | `/security-scan`        | Deploy (CRITICAL) | HARDEN              |
| Perf audit        | `/perf-audit`           | Report only       | HARDEN              |
| Build             | `npm run build`         | Deploy            | DEPLOY              |
| Smoke test        | `/smoke-test $URL`      | Production        | DEPLOY              |

---

## CI/CD Pipeline

```
git commit   → husky: lint → type-check → test (3-gate pre-commit)
git push     → GH Actions ci.yml: lint + type-check + test + build (parallel)
PR to main   → ci.yml (same)
staging      → deploy-staging.yml: CI + migrations + Render deploy + smoke test
main merge   → deploy-production.yml: CI + manual approval + migrations + Render + GitHub Release
```

---

## Docs Structure

```
docs/
├── pm/             ← SPRINT_STATE.md (PM brain — session state)
├── discovery/      ← /discover outputs (one file per feature)
├── api/            ← /api-docs outputs (one file per feature)
├── harden/         ← /harden outputs (security + perf reports)
├── sprints/        ← /sprint-plan + /sprint-report + /deploy-check outputs
└── guides/         ← Development guides
```

---

## Project Conventions

| Convention       | Rule                                                                           |
| ---------------- | ------------------------------------------------------------------------------ |
| Commits          | `feat:`, `fix:`, `chore:`, `docs:`, `test:`, `refactor:`, `perf:`, `security:` |
| Branches         | `feature/name`, `fix/name`, `sprint/sprint-N`                                  |
| Shared types     | Define in `packages/shared/src/types/` — never duplicate                       |
| Input validation | Zod schemas on all API inputs                                                  |
| Soft deletes     | `deleted_at TIMESTAMPTZ` — never `DELETE FROM` user data                       |
| Test fixtures    | Proper UUIDs — never `'user-1'` shorthand                                      |
| Migrations       | Apply via Supabase MCP, then regen types                                       |
| Session end      | Always update `docs/pm/SPRINT_STATE.md` handoff section                        |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jp8617465-sys)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Jp8617465-sys)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
