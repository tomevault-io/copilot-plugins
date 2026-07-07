## f1fantasyapp

> Full-stack F1 Fantasy Sports application combining React frontend and .NET backend.

# F1 Fantasy App Monorepo

Full-stack F1 Fantasy Sports application combining React frontend and .NET backend.

## System Overview

F1 Fantasy Sports platform where users build fantasy F1 teams, join leagues, and earn points based on real race performance.

**Architecture:**

```
React SPA (Vite) → .NET 10 Minimal API → PostgreSQL
                ↓
            Supabase Auth
```

**Tech Stack:**

- **Frontend:** React 19, TypeScript, TanStack Router, Tailwind CSS v4, shadcn/ui
- **Backend:** .NET 10 ASP.NET Core Minimal API, Entity Framework Core
- **Database:** PostgreSQL
- **Services:** Supabase (authentication), Sentry (monitoring)

## Domain & Features

### Core Concepts

- **Team** — Each user creates one team per season. Teams are subject to a budget cap; each driver/constructor has a price and the projected spend cannot exceed the cap.
- **Roster Lock** — Each race has a `LockDeadline`. Once `now >= lockDeadline`, the team can no longer be modified (drivers/constructors cannot be added or removed). The UI shows a live countdown and disables pickers when locked.
- **League** — Users create or join leagues to compete against others. Public leagues are browsable; private leagues use invite tokens. Max 15 teams per league. A team can belong to multiple leagues.
- **Scoring** — Teams earn points based on real F1 race results. See `docs/research/fantasy-rules/decisions/scoring.md` for the rules.
- **Season / Race** — Seasons map to F1 calendar years and contain ordered races (with round numbers and lock deadlines). Driver and constructor pricing is dynamic per season.

## F1 Domain

**Grid:** 22 drivers across 11 constructors. Each constructor fields exactly 2 drivers.

**Race weekends** come in two formats as it pertains to this game:

- **Standard:** Qualifying → Race
- **Sprint** (~6 per season): Sprint → Qualifying → Race

**Game rules and design decisions** are documented in `docs/research/fantasy-rules/decisions/`:

- `design-goals.md` — Player experience goals that all other decisions are evaluated against
- `format.md` — Team shape: slot counts, budget cap, composition constraints
- `rules.md` — Gameplay mechanics: transfers, captain, locking, budget, edge cases
- `scoring.md` — Point tables and scoring logic (intentionally diverges from official F1 Fantasy)
- `pricing.md` — Preseason pricing source, in-season PPM-based price movement formula

The official F1 Fantasy scoring rules are captured for reference in `docs/research/fantasy-rules/reference/f1-official-scoring.md`.

## Project Context

- **Team Size**: Solo developer
- **Development Philosophy**: Balance simplicity with proper patterns - avoid both over-engineering for scale and shortcuts that create technical debt
- **F1 Season**: The F1 season aligns with the calendar year. When referencing teams, drivers, regulations, or example data, use the current season's information first, falling back to the previous season only when current-season data isn't available.

## Claude Code Preferences

- Avoid over-engineering; keep solutions focused on the requested task
- Adhere to YAGNI philosophy
- When in doubt about approach, ask rather than proceed
- Keep solutions focused on solving the cause of a problem, not the symptom
- Use conventional commit styling for commit messages

## Feature Planning

When planning features (via plan mode or when asked to plan), organize the plan into a sequence of **self-contained commits**. Each commit is a gate — wait for user approval before moving to the next one.

- Each commit includes **both functionality and its tests** — never split implementation from tests.
- Each commit must independently pass build, lint, tests, and formatting.
- Commits should be iterative and incremental — each builds on the last.
- Order commits so earlier ones lay the foundation (e.g., data model before API before frontend).
- Keep commits focused. If a commit is doing too much, split it further.
- Write the plan to `docs/plans/` when producing a full plan.

## Testing Strategy

Three test types, each owning failure modes only it can see. Coverage is measured in failure modes, not scenarios — overlap across layers is fine when each layer catches something the others can't.

### What each layer owns

**Unit tests** — pure logic, in-process, no I/O.

- Own: branches, edge cases, error paths, mappings, calculations, validation rules.
- Goal: full combinatorial coverage of logic. Millisecond feedback.
- Dependencies: real collaborators within the process; inject clock/RNG/IDs for determinism.

**Integration tests** — one or more real boundaries crossed (DB, HTTP pipeline).

- Own: SQL correctness, EF config, migrations, serialization, auth/middleware, model binding, transaction behavior, endpoint contracts.
- Prefer the HTTP seam (`WebApplicationFactory`) over calling handlers directly — covers the pipeline for free.
- Use real Postgres via Testcontainers. **Never** EF InMemory or SQLite-for-Postgres — they diverge from prod on exactly what these tests exist to verify.
- Isolate per test (transaction rollback or truncate). No shared fixtures.
- Stub third parties you don't own (Supabase, Sentry). Don't stub your own code.

**E2E tests** — full stack through a real browser.

- Own: cross-system wiring — CORS, cookie/auth flows, deployment/config, build output, client/server contract drift, critical user journeys.
- Keep the suite small: happy paths of core flows (sign-in, build team, join league, view scores).
- Semantic selectors (`data-testid`, role, accessible name). Never CSS structure.
- Seed via API/DB, not through the UI. Proper waits, never `sleep`.

### Which test to reach for

Ask: **"What's the smallest test that could catch this bug?"** Write that one.

| Change                                                             | Primary layer         |
| ------------------------------------------------------------------ | --------------------- |
| Business rule, calculation, mapping, validation                    | Unit                  |
| New/changed SQL, EF query, migration                               | Integration (real DB) |
| New endpoint, auth rule, middleware, model binding                 | Integration (HTTP)    |
| React component rendering, state, user interaction                 | Unit (component test) |
| Frontend routing + data fetching with mocked API                   | Frontend integration  |
| New user-facing flow, auth/session wiring, deploy-affecting config | E2E                   |

### Overlap rules

Overlap is correct when each layer catches a distinct failure mode. It's waste when it doesn't.

- Cover branch logic at the **lowest** layer that can see it. Don't re-walk the same matrix higher up.
- Higher layers get **one happy path** per flow, plus one representative failure only if that interaction is load-bearing (e.g., roster-locked message, auth redirect).
- If an integration test passes/fails in lockstep with an E2E on the same path with the same assertions, one of them is redundant — keep the faster one unless the slower one proves something the faster can't.

### Anti-patterns

- **In-memory DB substitutes** for code that ships on Postgres.
- **Mocking your own code** inside an integration test, or mocking third parties inside a unit test just to mock them.
- **1:1 test-per-method** structure — encodes implementation into the suite; every refactor becomes a rewrite.
- **Validation/edge-case matrices in E2E** — belongs in unit.
- **Shared mutable fixtures** ("the one big seed") — tests become order-dependent and nobody dares change the fixture.
- **`sleep`-based waits** in async or E2E tests.
- **Pixel-perfect screenshot diffing** as regression safety.
- **Coverage % as a goal** — high coverage with weak assertions resists refactoring and gives false confidence.
- **Ice-cream cone** (many E2E, few unit) — slow CI, flaky, failures are hard to localize.

### Practical defaults

- Backend unit tests: xUnit, no DB, no `WebApplicationFactory`, milliseconds each.
- Backend integration tests: `WebApplicationFactory` + Testcontainers Postgres. Transaction-rollback isolation.
- Frontend unit/component tests: Vitest + `@testing-library/react`.
- E2E: Playwright, small suite, runs against a prod-like build. Parallel workers with isolated users.

## Manual verification

Some changes have run-time-only payoff that tests don't surface; those warrant a browser check against the dev stack.

- **Authed flows:** sign up a throwaway user — the confirmation code lands in the local Supabase mail catcher.
- **Signed-out behavior:** use an isolated browser context so an existing session stays intact.

## Commenting Strategy

Default to no comment. Every comment is debt — it can rot, mislead, or distract. Write one only when the alternative is worse: a reader would otherwise misunderstand the code or spend time discovering something the comment encodes directly.

### The test before writing

Ask: **"Would removing this confuse a competent reader six months from now, reading the code cold?"** If no — cut it. Trust the names, the types, and the structure to carry the meaning.

### When a comment earns its place

- **Non-obvious WHY.** A constraint or invariant that wouldn't show up in the AST. _"Caller must hold the write lock." "API returns 200 with an error body for this case — peek at the JSON."_
- **A surprise.** Behavior a reader would assume is a bug. _"Off-by-one is intentional — F1 rounds are 1-indexed." "We retry on 401 — auth service emits them during clock skew."_
- **Workaround pointer.** External constraint forcing a non-obvious shape. _"Chrome bug crbug.com/123456 — drop when our floor is M120."_
- **Domain pointer.** Spec/RFC/paper that explains the rule. _"RFC 7231 §6.4.3: 303 always GETs the target."_

### When a comment doesn't

- **What the code or types already say.** `// Iterate users` above a `for` loop; `// Returns 'expired' or null` above `(): Promise<'expired' | null>`. Rename, don't comment.
- **Task context.** _"Fix for #164." "Per the design doc."_ Belongs in the commit message; rots when issues close.
- **Historical state.** _"Was synchronous before."_ `git log` owns this; the file describes what is, not what was.
- **Internal callers / provenance.** _"Used by SignUpForm." "Set by indexRoute.beforeLoad."_ Find-references gives you this for free; comment goes stale on the next caller.
- **Design rationale.** _"We chose route context over AuthContext because…"_ Lives in `docs/plans/`, not the source file. The code is the chosen path; the argument is dead weight once the choice is made.
- **Reassurance about performance/correctness of an implementation detail.** _"This is cached, so it's free to call."_ Reader doesn't need it.

### Heuristics

- Tempted to write _"// this X-es the Y"_ → rename instead.
- Tempted to write _"// because Z"_ → is Z visible in the code (keep) or only in your head / the conversation / the plan (cut).
- Watch the diff-talk smell: _"now uses," "still works," "originally was"_ narrate history, not state.
- Read it cold. If a reader needs to know "the resend flow" or "the X migration" to parse the comment, rewrite in plain terms or delete.

## Git Commit Message Preferences

- Do not include the "Generated with Claude Code" footer in commit messages or PR descriptions

## Repository Structure

- `web/` - React/TypeScript frontend with Vite (see web/CLAUDE.md)
- `api/` - .NET 10 ASP.NET Core API backend (see api/CLAUDE.md)
- `e2e/` - Playwright browser suite against a prod-like local build (see e2e/README.md)

## Quick Start

Always use `npm run <script>` from the repo root — never `npx`, `cd api && dotnet`, or `cd web && npm` directly.

### Development

```bash
npm run web:install  # Install frontend dependencies (run once)
npm run web:dev      # Frontend at http://localhost:5173
npm run api:watch    # API with hot reload (preferred over api:run)

# Or use VSCode tasks: "Start All Servers"
```

### Testing

```bash
npm run test:all               # Frontend + backend (unit + integration). Does not run e2e.
npm run web:test               # Frontend tests
npm run web:test:watch         # Frontend tests in watch mode
npm run web:coverage           # Frontend test coverage
npm run api:test               # Backend unit + integration (Testcontainers Postgres; Docker required)
npm run api:test:unit          # Backend unit tests only
npm run api:test:integration   # Backend integration tests only
npm run e2e:install            # Install Playwright deps + Chromium (run once)
npm run e2e                    # E2E suite (requires `cd e2e/supabase && supabase start` first)
npm run e2e:ui                 # Playwright UI mode for debugging
```

### Building

```bash
npm run web:build  # Build frontend
npm run api:build  # Build API
```

### Code Quality

```bash
npm run web:lint          # Lint frontend
npm run web:format        # Format frontend
npm run web:format:check  # Check frontend formatting (CI)
npm run api:format        # Format backend
npm run api:format:check  # Check backend formatting (CI)
```

## VSCode Integration

Open this folder in VSCode and use:

- **Tasks** (Cmd+Shift+P → "Tasks: Run Task")
  - "Start All Servers" - Launches both dev servers
  - "[Web] Dev Server" - Frontend only
  - "[API] Watch" - Backend only
  - "Build All" - Full build
- **Debugging** (F5)
  - "Full Stack (Web + API)" - Debug both simultaneously

## Local Services Topology

See [Local Services Topology](README.md#local-services-topology) in the README for the dev vs e2e stack layout, port-shift rule, and migration-sharing details. Quick recall: dev processes use Supabase `54321–54324` / web `5173` / API `5077`; e2e processes are all shifted by +100; `api/supabase/migrations/` is the source of truth and e2e symlinks to it.

## Production Infrastructure

Hosted on Fly.io + Supabase (free tier).

| Resource                | Identifier                            |
| ----------------------- | ------------------------------------- |
| Fly.io app name         | `f1fantasyapp`                        |
| Fly.io region           | `iad` (Virginia)                      |
| Supabase project ref    | `cfuccajsckqzecbfyqrv`                |
| Supabase direct DB host | `db.cfuccajsckqzecbfyqrv.supabase.co` |

**MCP servers available for investigation:**

- `fly logs -a f1fantasyapp` — runtime logs from the API
- `mcp__sentry__search_events` — error events (project slug: `f1-fantasy-api` or `f1-fantasy-web`, org: `emsqrd`, regionUrl: `https://us.sentry.io`)
- `mcp__supabase__get_logs` — Postgres and API gateway logs (service: `postgres` or `api`)

**Initial page load (authed, at `/`):** `GET /api/me/profile` fires in parallel with `GET /api/seasons/current`, then `GET /api/me/team/summary` + `GET /api/me/standings` + `GET /api/seasons/{id}/race-weekends` fire together. Anonymous visits fetch nothing.

## Project Documentation

- `web/CLAUDE.md` - Frontend architecture, patterns, and conventions
- `api/CLAUDE.md` - Backend architecture, patterns, and conventions
- `api/F1CompanionApi.IntegrationTests/README.md` - Backend integration test fixture + auth helpers
- `e2e/README.md` - Playwright suite: prerequisites, architecture, selector discipline
- `docs/research/` - Research findings and design specs
- `docs/mockups/` - Static HTML mockups (self-contained, design tokens from `web/src/index.css`)
- `docs/plans/` - Feature implementation plans (written during plan mode)

## Agent skills

### Issue tracker

GitHub issues in `emsqrd/f1fantasyapp` via the `gh` CLI. See `docs/agents/issue-tracker.md`.

### Issue & PR writing

Title and body conventions for issues and PRs; create new issues labeled `needs-triage`. See `docs/agents/issue-pr-style.md`.

### Triage labels

Canonical role names used as-is (`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`). See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: `CONTEXT.md` + `docs/adr/` at the repo root. See `docs/agents/domain.md`.

---
> Source: [emsqrd/f1fantasyapp](https://github.com/emsqrd/f1fantasyapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
