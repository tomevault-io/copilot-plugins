## mini-loan-management-system

> Mini Loan Management System — a demo of AI-native SDLC for the pre-sales engineering team.

# CLAUDE.md — Mini Loan Management System

## Project Context

Mini Loan Management System — a demo of AI-native SDLC for the pre-sales engineering team.
Every feature in this repo was built using the process documented in `docs/04-workflows/`.
Goal: show clean architecture + modern tooling + AI-assisted development end-to-end.

## Stack

- **Backend**: ASP.NET Core 9 · EF Core 9 · SQL Server · FluentValidation · AutoMapper · JWT · Serilog
- **Frontend**: Angular 19 · Angular Material · Signals · Standalone Components · pnpm
- **Testing**: xUnit (75% coverage gate) · Jasmine/Karma
- **CI/CD**: GitHub Actions · Claude Sonnet automation
- **Tools**: .NET SDK 9.0.x (global.json) · Node 20.x (Volta) · pnpm 9.x (Volta) · Docker

## Monorepo Structure

See `docs/00-overview/architecture.md` for the full annotated tree.

**Backend pattern**: single project, folder-organised (N-Tier / CrisMAC AXIS).
Call flow: `Controller → Service → Repository → EF Core DbContext`
No cross-folder circular references. Services do not call other services directly — orchestrate via the controller if needed.

**Frontend**: Angular 19 workspace, two projects: `loan-app` (main app) + `shared-lib` (guards, interceptors, models).

## Naming Conventions

### Backend (C#)

- PascalCase: classes, methods, properties, interfaces (`ILoanRepository`)
- camelCase: local variables, parameters
- Async suffix on all async methods (`GetLoanByIdAsync`)
- No MediatR/CQRS — no Commands, Queries, or Handlers

### Frontend (TypeScript/Angular)

- PascalCase: components, services, pipes, classes
- camelCase: methods, properties, variables
- kebab-case: file names, CSS classes, route paths
- Signals: plain noun (`loans`, `currentUser`, `isLoading`)
- Computed signals: derived prefix or adjective (`filteredLoans`, `isAdmin`)
- Services: `[Feature]Service` · Guards: `[Role/Feature]Guard`

## API Design

### HTTP path naming (mandatory, project-agnostic)

All REST URL paths MUST follow these rules, summarized from [Good routes versus bad routes](https://thuongthanhto.medium.com/good-routes-versus-bad-routes-e7e70dfae1bf) (Thuong To, Medium):

1. **Lowercase paths** — path segments use lowercase; separate words with **hyphens** (not underscores, camelCase, or Title Case in the path). Avoid abbreviated segment names; use full, meaningful words. **Path parameter names** use **camelCase** inside `{ }` (e.g. `{loanId}`).
2. **Hierarchy with `/`** — use forward slashes only for parent/child relationships (never backslashes).
3. **Nouns, plural collections** — resource names are **nouns**, not verbs; use **plural** nouns for collections. Express actions with HTTP methods (and bodies) rather than verb-shaped path segments where possible.
4. **No special characters** in paths — avoid punctuation and symbols in fixed segments (documented exceptions only, e.g. multi-value rules in query strings).
5. **No file extensions** in the URI — do not use `.json` / `.xml` on paths; use the `Accept` header or a query parameter for representation format.
6. **Query string for filters and paging** — filtering, sorting, page size, and page number belong in **query parameters**, not extra path segments.
7. **No trailing slash** — do not publish or document paths with a trailing `/`.

### This repository

- Base path: `/api/v1/`
- Auth: Bearer JWT in `Authorization` header
- Success: `{ data: T, message: string, timestamp: string }`
- Errors: ProblemDetails (RFC 7807) — never leak stack traces
- Versioning: URL path (`/api/v1/`, `/api/v2/`)

## Database Conventions

- Table names: PascalCase (`Users`, `Loans`, `RepaymentSchedules`)
- Migrations: numbered prefix — `001_InitialSchema`, `002_AddLoanIndex`
- EF Core Code-First, Fluent API for all configurations (**no DataAnnotations on entities**)
- No raw SQL — EF Core LINQ queries only

## Testing Standards

- Coverage target: **75% (hard gate in CI)**
- Unit test focus: Services, EMI calculation helper, validators
- Test naming: `MethodName_Scenario_ExpectedResult`
- No mocking of EF Core — use InMemory provider for service tests
- Frontend: test services and signal state, not template rendering

## Workflow Orchestration

### Plan Mode (MANDATORY)

Enter plan mode for ANY task with 3+ steps or architectural impact.
If something breaks mid-task: STOP, re-plan, do not push through.
Use plan mode for verification steps too — not just building.

### Subagent Strategy

Offload research and parallel exploration to subagents.
Keep main context window clean — one subagent per concern.
Use Explore subagent for codebase navigation.
Use Plan subagent for architecture decisions.

### Self-Improvement Loop

After ANY user correction: update `tasks/lessons.md` immediately.
Write a rule that prevents the same mistake.
Review `tasks/lessons.md` at the start of each session on this project.

### Verification Before Done

Never mark a task complete without running the code.
Run tests, check build output, confirm correctness.
Ask: "Would a senior engineer approve this?"

## Task Management

- Write plan to `tasks/todo.md` with checkable items before starting
- Check in with user before implementation begins on non-trivial tasks
- Mark items complete as you go
- Add a review section to `tasks/todo.md` after completing a feature
- Capture lessons in `tasks/lessons.md` after corrections

## Git Workflow

**Permanent branches (NEVER commit to directly):** `main` · `develop` · `release` · `uat`

**Promotion flow:** `develop → release → uat → main`

### Branch Naming

All short-lived branches follow `{issue-number}-kebab-case-description`.
No `feature/`, `fix/`, `hotfix/`, or type-prefix folders — ever.

**Rules:**

- Start with the GitHub issue number: `28-fix-login-token`
- Lowercase, hyphens only — no underscores, no uppercase
- Short and descriptive — core intent, under ~6 words, skip filler words
- No trailing hyphens, no double hyphens

**Examples:**

| Issue                               | Branch name                |
| ----------------------------------- | -------------------------- |
| #28 Fix login token refresh error   | `28-fix-login-token`       |
| #45 Add OTP screen to login page    | `45-add-otp-screen`        |
| #91 Remove legacy payment endpoints | `91-remove-legacy-payment` |

### Branch Creation Workflow (MANDATORY)

Claude must never create a local branch and push it. The required sequence is:

1. **Create a GitHub issue** — title describes the work; assign to the developer or `@claude`
2. **Create the remote branch from the issue** — use `gh issue develop {number} --base develop --checkout`
   This creates `{number}-kebab-description` on the remote and checks it out locally
3. **All commits go on that branch** targeting `develop`

> Never do `git checkout -b` without a GitHub issue first.

### Pull Request Process

1. **Rebase before opening PR:**
   ```bash
   git checkout develop && git pull origin develop
   git checkout {branch} && git rebase develop
   ```
2. **Create PR with `gh pr create`:**
   - Title: clear and concise, mirrors the branch purpose
   - Description: what and why; use `Fixes #N` / `Resolves #N` to auto-close the issue
   - Assignee: self (or Claude)
   - Reviewers: set per rotation or expertise
3. **Do not DM reviewers** — the PR notification is sufficient
4. **Squash on merge** — delete intermediate commit messages; write one clean final commit following Conventional Commits format

### Conventional Commits (enforced by commitlint)

```
feat(scope):     new feature
fix(scope):      bug fix
docs(scope):     documentation only
style(scope):    formatting, no logic change
refactor(scope): no behaviour change
perf(scope):     performance improvement
test(scope):     tests only
build(scope):    build system or dependencies
ci(scope):       CI/CD changes
chore(scope):    tooling, config
revert(scope):   revert a previous commit
```

Commit title: ≤ 50 chars · imperative mood · no trailing period
Body: wrap at 72 chars · explain what and why · reference issue (`Related to #N`)
Do not put issue numbers in the title.

**Scope:** required on every commit (commitlint `scope-empty: never`). Use a short, meaningful scope for the area touched — not limited to a fixed list.

**Scopes often used in this repo (reference only):** `auth` · `loans` · `repayments` · `admin` · `borrower` · `shared` · `infra` · `ci` · `docs` · `deps` · `root`

All feature work targets `develop`. `main` is production — merged only after UAT sign-off via PR.
Claude creates PRs automatically — humans approve merge.
Run `scripts/validate-branch.js` via pre-push Husky hook.

## Core Principles

1. **Document first** — write ADR or update `docs/` before coding a new pattern
2. **Plan mode for 3+ step tasks** — no ad-hoc changes to architecture
3. **TDD** — write test first, make it pass, then refactor
4. **No TODO comments in committed code** — use `tasks/todo.md` instead
5. **No magic numbers** — define constants in `Constants/AppConstants.cs`
6. **Minimal impact** — only change what the task requires
7. **Simplicity first** — three clear lines over a premature abstraction

## Off-Limits (Do NOT Do)

- Do not add features not in the current task
- Do not use `--no-verify` to bypass git hooks
- Do not commit directly to `develop`, `release`, `uat`, or `main`
- Do not hardcode connection strings or JWT secrets
- Do not use DataAnnotations on domain entities (use FluentValidation / EF Fluent API)
- Do not use MediatR, CQRS, Commands, Queries, or Handlers — this is N-Tier, not Clean Architecture

## Future Scope (Do Not Implement Now)

- OAuth2 / SSO authentication
- External API integrations
- Data pipeline / ETL processes
- ML integration
- Multi-tenancy

---
> Source: [aditya-shinde-d2k/mini-loan-management-system](https://github.com/aditya-shinde-d2k/mini-loan-management-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
