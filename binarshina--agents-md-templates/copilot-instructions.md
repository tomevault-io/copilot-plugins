## agents-md-templates

> > Template for AI coding agents. Replace every placeholder in `<angle_brackets>` with real project data before adopting this file. Remove example rows you do not need, but prefer filling sections over deleting them. This file should be concrete, current, and operational.

# AGENTS.md

> Template for AI coding agents. Replace every placeholder in `<angle_brackets>` with real project data before adopting this file. Remove example rows you do not need, but prefer filling sections over deleting them. This file should be concrete, current, and operational.

---

## Purpose

This file is the primary operating manual for AI agents working in this repository.

It should give the agent enough context to:

- understand what the project does;
- follow the correct architecture and conventions;
- place code in the right directories;
- run the right commands for setup, validation, and testing;
- avoid unsafe or low-quality changes;
- know when to stop and ask a human.

If this file is vague, outdated, or contradictory, agent output will degrade. Keep it specific.

---

## Project Snapshot

- Project name: `<project_name>`
- Project type: `<web app | api | mobile app | cli | library | monorepo | service | internal tool | other>`
- One-line description: `<what this project does in one sentence>`
- Primary users: `<who uses this system>`
- Business/domain context: `<domain, workflow, or problem space>`
- Lifecycle stage: `<prototype | mvp | production | mature | legacy modernization>`
- Maintainers / owning team: `<team_name_or_people>`
- Default branch: `<main_branch_name>`
- Repo status notes: `<active refactor | migration in progress | legacy areas | frozen modules | none>`

### Fill-In Guidance

- Write the description for an engineer, not for marketing.
- If the system is legacy or mid-migration, say so explicitly.
- If the repository contains multiple products or apps, mention that here and define the structure below.

### Example

```md
- Project name: Acme Orders
- Project type: Monorepo
- One-line description: Internal platform for creating, pricing, and tracking wholesale orders.
- Primary users: Sales operators, finance team, warehouse integrations
- Business/domain context: B2B commerce
- Lifecycle stage: Production
- Maintainers / owning team: Orders Platform Team
- Default branch: main
- Repo status notes: Legacy pricing flow is being migrated into feature-owned modules
```

---

## Agent Principles

Unless the user explicitly asks otherwise, the agent should:

- prefer the smallest safe change that solves the task;
- preserve existing architecture and naming conventions;
- update tests when behavior changes;
- update docs, config, or examples when they become stale because of the change;
- verify work before finishing;
- avoid speculative refactors;
- ask before destructive, irreversible, expensive, or production-affecting operations.

### Optimize For

1. Correctness
2. Maintainability
3. Speed

### Never Do These By Default

- Rewrite architecture without being asked.
- Introduce a new dependency when an existing project dependency can solve the problem.
- Manually edit generated files if the intended workflow is regeneration.
- Ignore failing checks related to the files or behavior you changed.
- Guess around security-sensitive, billing-sensitive, or compliance-sensitive behavior.

---

## Sources Of Truth

Consult these references before making non-trivial changes:

| Source | Path / URL | When To Use It |
| --- | --- | --- |
| Product or domain docs | `<path_or_url>` | `<behavior rules / business logic>` |
| Architecture docs | `<path_or_url>` | `<module boundaries / system design>` |
| ADRs / decisions log | `<path_or_url>` | `<why previous tradeoffs were made>` |
| API contracts / schemas | `<path_or_url>` | `<endpoint, event, or schema changes>` |
| Design system / UI docs | `<path_or_url>` | `<UI behavior, components, tokens>` |
| Contribution guide | `<path_or_url>` | `<repo workflow, branching, review expectations>` |
| Security docs | `<path_or_url>` | `<auth, secrets, permissions, compliance>` |
| Deployment / runbooks | `<path_or_url>` | `<environment, release, infrastructure changes>` |

If documentation and code disagree, prefer `<code | docs | ask_human>` and mention the mismatch in your final summary.

### Example

```md
| Source | Path / URL | When To Use It |
| --- | --- | --- |
| Architecture docs | `docs/architecture.md` | Changes to module boundaries or shared packages |
| ADRs / decisions log | `docs/adr/` | New abstractions, workflow changes, tradeoff decisions |
| API contracts / schemas | `openapi/openapi.yaml` | Any endpoint or generated client change |
```

---

## Tech Stack

Do not write “latest”. Use exact versions or supported ranges.

### Core Stack

- Language(s): `<language_and_versions>`
- Runtime(s): `<runtime_and_versions>`
- Framework(s): `<frameworks_and_versions>`
- Package manager(s): `<package_managers_and_versions>`
- Build tool(s): `<build_tools_and_versions>`
- Database(s): `<db_and_versions>`
- Messaging / queueing: `<tool_or_none>`
- Cache / storage: `<tool_or_none>`
- Hosting / infrastructure: `<cloud | on-prem | hybrid | other>`

### Key Libraries And Services

List the important tools the agent should recognize immediately:

| Area | Library / Service | Version | Purpose | Notes / Constraints |
| --- | --- | --- | --- | --- |
| `<frontend/backend/data/auth/etc>` | `<name>` | `<version>` | `<why it exists>` | `<special rules>` |
| `<frontend/backend/data/auth/etc>` | `<name>` | `<version>` | `<why it exists>` | `<special rules>` |
| `<frontend/backend/data/auth/etc>` | `<name>` | `<version>` | `<why it exists>` | `<special rules>` |

### Version Policy

- Required versions: `<exact_versions_or_ranges>`
- Version source of truth: `<package.json | pyproject.toml | go.mod | Gemfile | Dockerfile | etc>`
- Dependency update policy: `<manual | renovate | dependabot | scheduled>`
- Compatibility requirements: `<browser matrix | runtime matrix | API compatibility | DB compatibility>`

### Example

```md
- Language(s): TypeScript 5.6
- Runtime(s): Node.js 22.x
- Framework(s): Next.js 15, React 19
- Package manager(s): pnpm 10
- Build tool(s): Turborepo 2
- Database(s): PostgreSQL 16
- Messaging / queueing: none
- Cache / storage: Redis 7, S3
- Hosting / infrastructure: AWS
```

---

## Architecture

- Architecture style: `<feature-sliced | layered | clean architecture | hexagonal | modular monolith | microservices | event-driven | other>`
- High-level description: `<how the system is divided and why>`
- Main modules / bounded contexts: `<module_a, module_b, module_c>`
- Main data flow: `<request/event/render flow>`
- State management approach: `<if relevant>`
- Integration boundaries: `<external APIs, internal services, SDKs, jobs, queues>`
- Areas under migration: `<legacy_to_new transitions>`
- Hard constraints: `<what must not be broken>`

### Architectural Rules

- Put `<kind_of_logic>` in `<allowed_path_or_layer>`, not in `<forbidden_place>`.
- Keep `<module>` independent from `<module>`.
- New work in `<area>` must follow `<pattern>`.
- Reuse abstractions from `<paths>` before creating new ones.
- Do not bypass `<validation | domain layer | auth layer | schema boundary>` for convenience.

### Fill-In Guidance

- If you name an architecture style, explain what it means in this repo.
- Mention real boundaries and anti-patterns, not just textbook terms.
- Call out migration zones explicitly. Agents often break these if not warned.

### Example

```md
- Architecture style: Modular monolith with feature-oriented boundaries
- High-level description: Each business capability owns its API layer, application logic, domain rules, and persistence adapters
- Main modules / bounded contexts: orders, pricing, customers, invoicing
- Main data flow: request -> route -> application service -> domain -> repository -> response mapper
- Areas under migration: pricing logic is being moved out of shared utils into the pricing module
- Hard constraints: customer state transitions must remain backward compatible with existing warehouse integrations
```

---

## Repository Structure

Document the tree as if onboarding a new engineer on day one.

```text
<repo-root>/
├─ <dir-or-file>/        # <what it contains>
├─ <dir-or-file>/        # <what it contains>
├─ <dir-or-file>/        # <what it contains>
├─ <dir-or-file>/        # <what it contains>
└─ <dir-or-file>/        # <what it contains>
```

### Directory Responsibilities

| Path | Responsibility | Typical Contents | Must Not Contain |
| --- | --- | --- | --- |
| `<path>` | `<purpose>` | `<examples>` | `<anti_examples>` |
| `<path>` | `<purpose>` | `<examples>` | `<anti_examples>` |
| `<path>` | `<purpose>` | `<examples>` | `<anti_examples>` |

### File Placement Rules

- New features go in `<path>`.
- Shared reusable code goes in `<path>`.
- One-off scripts go in `<path>`.
- Generated artifacts live in `<path>` and should `<be_committed_or_not>`.
- Env/config files live in `<path>`.
- Migrations, schemas, and contracts live in `<path>`.

### Example

```md
apps/web/          # customer-facing frontend
apps/admin/        # internal operations frontend
packages/ui/       # shared UI components and tokens
packages/domain/   # shared domain types and business rules
infra/             # deployment, IaC, environment templates
docs/              # architecture notes, ADRs, onboarding docs
```

---

## Environment Setup

### Required Tooling

- Required tools: `<tool_and_version>`, `<tool_and_version>`, `<tool_and_version>`
- Install dependencies: `<exact_command>`
- Start local environment: `<exact_command>`
- Start dependent services only: `<exact_command>`
- Seed / bootstrap data: `<exact_command>`
- Load environment variables from: `<path>`
- Required local services: `<db | docker | queue | emulator | none>`

### Setup Notes

- State any required command order.
- State whether Docker is required or optional.
- State whether seeds are safe to run repeatedly.
- State whether any manual credentials or local certificates are required.

### Example

```md
- Required tools: Node.js 22.x, pnpm 10, Docker Desktop
- Install dependencies: pnpm install
- Start local environment: pnpm dev
- Start dependent services only: docker compose up -d
- Seed / bootstrap data: pnpm db:seed
- Load environment variables from: .env.local
- Required local services: Postgres, Redis
```

---

## Development Commands

Every command below should work as written.

| Task | Command | Scope | Notes |
| --- | --- | --- | --- |
| Install dependencies | `<command>` | `<repo_or_package>` | `<notes>` |
| Start development | `<command>` | `<repo_or_package>` | `<notes>` |
| Start one service/package | `<command>` | `<repo_or_package>` | `<notes>` |
| Build | `<command>` | `<repo_or_package>` | `<notes>` |
| Lint | `<command>` | `<repo_or_package>` | `<notes>` |
| Format | `<command>` | `<repo_or_package>` | `<notes>` |
| Typecheck | `<command>` | `<repo_or_package>` | `<notes>` |
| Run all tests | `<command>` | `<repo_or_package>` | `<notes>` |
| Run one test file | `<command>` | `<repo_or_package>` | `<notes>` |
| Run one test case | `<command>` | `<repo_or_package>` | `<notes>` |
| Run integration tests | `<command>` | `<repo_or_package>` | `<notes>` |
| Run e2e tests | `<command>` | `<repo_or_package>` | `<notes>` |
| Regenerate code | `<command>` | `<repo_or_package>` | `<notes>` |

### Verification Strategy

The agent should usually validate changes in this order:

1. file-level or test-level checks;
2. nearest package or module checks;
3. full-repo checks when necessary;
4. release-grade checks before merge for risky or broad changes.

### Example

```md
| Task | Command | Scope | Notes |
| --- | --- | --- | --- |
| Lint | `pnpm lint` | repo | Runs all package linters |
| Typecheck | `pnpm typecheck` | repo | Must pass before merge |
| Run one test file | `pnpm vitest run src/features/orders/order-form.test.ts` | package | Fastest local verification |
```

---

## Testing Guide

- Test framework(s): `<frameworks>`
- Unit test location(s): `<paths>`
- Integration test location(s): `<paths>`
- E2E test location(s): `<paths>`
- Contract test location(s): `<paths_or_none>`
- Naming pattern(s): `<*.test.ts | test_*.py | etc>`
- CI workflow location: `<path>`

### Testing Rules

- Every behavior change should be backed by tests unless there is a documented reason not to.
- Bug fixes should include a regression test when practical.
- Prefer focused tests during iteration.
- Run broader suites when changing shared code, persistence, contracts, infrastructure, or sensitive workflows.
- Snapshot or golden updates must be reviewed, not blindly accepted.

### Test Matrix

| Test Type | Path / Scope | Command | When To Run |
| --- | --- | --- | --- |
| Unit | `<path>` | `<command>` | `<most logic changes>` |
| Integration | `<path>` | `<command>` | `<db/api/module-boundary changes>` |
| E2E | `<path>` | `<command>` | `<user workflow changes>` |
| Contract | `<path>` | `<command>` | `<schema/api/event changes>` |
| Performance | `<path>` | `<command>` | `<perf-sensitive changes>` |

### Example

```md
- Test framework(s): Vitest, Playwright
- Unit test location(s): src/**/*.test.ts
- E2E test location(s): tests/e2e/
- Naming pattern(s): *.test.ts
- CI workflow location: .github/workflows/ci.yml
```

---

## Code Style And Naming

- Formatter: `<tool>`
- Linter: `<tool>`
- Type policy: `<strict | moderate | dynamic with validation>`
- Comments policy: `<when comments are appropriate>`
- Import policy: `<absolute | relative | grouped | sorted>`
- Error handling style: `<exceptions | result objects | error wrappers | logging conventions>`
- Logging style: `<structured | plain text | tracing>`
- Configuration style: `<typed env schema | central config | other>`

### Naming Conventions We Prefer

| Item | Preferred | Avoid | Example |
| --- | --- | --- | --- |
| Files | `<preferred_style>` | `<avoid_style>` | `<example>` |
| Directories | `<preferred_style>` | `<avoid_style>` | `<example>` |
| Classes / components | `<preferred_style>` | `<avoid_style>` | `<example>` |
| Functions / methods | `<preferred_style>` | `<avoid_style>` | `<example>` |
| Variables | `<preferred_style>` | `<avoid_style>` | `<example>` |
| Constants | `<preferred_style>` | `<avoid_style>` | `<example>` |
| Types / interfaces / schemas | `<preferred_style>` | `<avoid_style>` | `<example>` |
| Test names | `<preferred_style>` | `<avoid_style>` | `<example>` |
| Branch names | `<preferred_style>` | `<avoid_style>` | `<example>` |

### Style Do / Don't

Do:

- use names that reflect intent;
- keep modules cohesive and purpose-driven;
- follow established patterns already used nearby;
- prefer explicit business logic over clever abstractions.

Don't:

- create “utils” dumping grounds for unrelated logic;
- mix multiple naming styles in the same area;
- hide important side effects behind vague helper names;
- introduce broad abstractions before a second real use case exists.

### Example

```md
| Item | Preferred | Avoid | Example |
| --- | --- | --- | --- |
| Files | kebab-case | mixedCase | order-summary-card.tsx |
| Classes / components | PascalCase | snake_case | OrderSummaryCard |
| Functions / methods | verbNoun | generic names | calculateOrderTotal |
| Test names | behavior-focused | “works correctly” | returns 422 when customer is archived |
```

---

## Preferred Patterns And Reference Implementations

Point the agent to real examples in the repository.

### Good Examples To Copy

- `<path>`: `<why this is a good example>`
- `<path>`: `<why this is a good example>`
- `<path>`: `<why this is a good example>`

### Patterns To Avoid Copying

- `<path>`: `<why it exists but should not be reused>`
- `<path>`: `<why it exists but should not be reused>`
- `<path>`: `<why it exists but should not be reused>`

### Fill-In Guidance

- Include examples for common work: feature modules, API handlers, tests, background jobs, migrations, UI components.
- Mark legacy examples clearly so the agent does not copy them into new code.

### Example

```md
- `src/features/orders/api/create-order.ts`: good validation -> service -> mapper flow
- `src/features/orders/components/order-form.tsx`: good example of feature-local UI composition
- `src/features/orders/order-form.test.ts`: good example of behavior-focused tests
- `src/legacy/shared/utils.ts`: legacy dumping ground, do not add new business logic here
```

---

## Data, Contracts, Codegen, And Migrations

- Schema location: `<path>`
- Migration location: `<path>`
- API contract location: `<path>`
- Event contract location: `<path_or_none>`
- Generated code location: `<path>`
- Regeneration command: `<command>`

### Rules

- Do not manually edit generated files if regeneration is the intended workflow.
- Preserve backward compatibility for `<api | events | database>` unless the task explicitly allows a breaking change.
- When changing contracts, also update tests, fixtures, docs, and dependent clients if applicable.
- For migrations, note whether they are reversible, destructive, or require rollout coordination.

### Example

```md
- Schema location: prisma/schema.prisma
- Migration location: prisma/migrations/
- API contract location: openapi/openapi.yaml
- Generated code location: src/generated/
- Regeneration command: pnpm codegen
```

---

## Security And Safety Boundaries

Treat this section as mandatory.

### Hard Rules

- Never commit secrets, private keys, access tokens, or production credentials.
- Never hardcode secrets in source code, tests, fixtures, or documentation.
- Redact sensitive values in logs and examples.
- Validate and sanitize untrusted input at the proper boundary.
- Use least privilege for database, cloud, and service credentials.
- Be extra careful in code touching auth, billing, PII, legal/compliance, infrastructure, or permissions.

### Human Approval Required Before

- deleting data or files;
- applying irreversible migrations;
- changing auth or permission logic;
- changing billing or payment flows;
- changing deployment or production infrastructure;
- installing or replacing major dependencies;
- rotating secrets or changing security configuration.

### Sensitive Areas

- Authentication / authorization: `<paths_or_notes>`
- Payments / billing: `<paths_or_notes>`
- Personal or regulated data: `<paths_or_notes>`
- Production configuration / infrastructure: `<paths_or_notes>`

---

## Git, PR, And Definition Of Done

- Branch naming convention: `<pattern>`
- Commit message convention: `<pattern>`
- PR title convention: `<pattern>`
- Changelog policy: `<when_and_how_to_update>`
- Release notes policy: `<if_applicable>`

### Definition Of Done

A change is not complete until:

1. relevant checks pass;
2. tests are added or updated where needed;
3. docs/config/examples are updated if affected;
4. file placement and naming follow this document;
5. assumptions, risks, and follow-up work are documented.

### Example

```md
- Branch naming convention: feat/<ticket>-<short-description>
- Commit message convention: conventional commits
- PR title convention: [ORD-123] Brief imperative summary
- Changelog policy: update CHANGELOG.md for user-visible changes
```

---

## Monorepo Guidance

If this repository contains multiple apps, packages, or services:

- the root `AGENTS.md` should define only global rules;
- each major app, package, or service should have its own nested `AGENTS.md`;
- the nearest `AGENTS.md` to the changed files should define local conventions;
- shared packages should document compatibility expectations and release constraints.

### Suggested Layout

```text
<repo-root>/
├─ AGENTS.md
├─ apps/
│  ├─ <app-a>/AGENTS.md
│  └─ <app-b>/AGENTS.md
├─ packages/
│  ├─ <shared-lib>/AGENTS.md
│  └─ <shared-ui>/AGENTS.md
└─ infra/
   └─ AGENTS.md
```

---

## Known Pitfalls

List real pitfalls the agent is likely to hit:

- `<pitfall_and_how_to_avoid_it>`
- `<pitfall_and_how_to_avoid_it>`
- `<pitfall_and_how_to_avoid_it>`

### Example

```md
- Do not import pricing helpers from `shared/utils`; use `modules/pricing` instead.
- Changes to `openapi/openapi.yaml` require regenerating typed clients.
- Legacy admin pages still rely on server-rendered templates; do not move them into the SPA without approval.
```

---

## When The Agent Must Stop And Ask

The agent should pause and ask a human when:

- requirements are ambiguous and there are multiple valid implementations;
- a change may break API compatibility, data compatibility, or deployment safety;
- documentation and code materially disagree;
- tests fail for reasons unrelated to the task and the cause is unclear;
- the task requires secrets, production access, or product-policy decisions;
- the safest path depends on a tradeoff the user has not chosen.

---

## Optional Cross-Tool Alignment

If this repository also uses tool-specific AI instruction files, keep them aligned:

- `README.md`
- `.github/copilot-instructions.md`
- `CLAUDE.md`
- `.cursorrules`
- `.aider.conf.yml`
- `.gemini/settings.json`

Prefer one authoritative source and mirror only the minimum necessary.

---

## Maintenance Checklist For Humans

- Update this file whenever the architecture, stack, commands, or workflow change.
- Keep commands executable exactly as written.
- Replace vague placeholders with real values before rollout.
- Add links to the best in-repo examples for common tasks.
- Split this file into nested `AGENTS.md` files when one file becomes too broad.

---

## Adoption Checklist

Before adopting this file for a real repository, confirm that you replaced:

- all placeholder values in `<angle_brackets>`;
- all example values copied from this template;
- all generic commands with real commands;
- all generic paths with real paths;
- all abstract rules with project-specific rules.

Before calling the file complete, confirm that it includes:

- project overview;
- stack and versions;
- architecture and boundaries;
- repository structure;
- exact setup/build/test commands;
- test locations and execution strategy;
- naming conventions;
- good and bad in-repo examples;
- security boundaries;
- escalation rules;
- links to source-of-truth documentation.

---
> Source: [binarshina/agents-md-templates](https://github.com/binarshina/agents-md-templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
