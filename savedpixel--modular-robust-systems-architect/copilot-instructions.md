## modular-robust-systems-architect

> Build and implement modular, robust software architecture for a specific feature, module, service, workflow, data model, integration, tool, API, package, or system boundary. Use only when the user wants architecture-guided implementation, modularization, hardening, refactoring, or integration work that needs questions, targeted project inspection, contracts, validation, migration safety, tests, and verification. Do not use for routine edits, isolated bug fixes, formatting, generic code advice, broad whole-project reviews, or global best-practice enforcement without a concrete target.


# Modular Robust Systems Architect

## Mission

Guide an agent to build and implement modular, robust software systems without overreaching across an entire project.

The skill exists for architecture-guided implementation. It helps an agent clarify the user's goal, inspect the relevant project evidence, design clean module boundaries, define contracts, implement incrementally, and verify the result with tests and operational checks.

Use this skill when the work needs durable architecture, not just code output. The expected result is a practical implementation path for a specific target that improves modularity and robustness while preserving existing working behavior.

## Bundled Resources

Load only the resource needed for the current task:

- `templates/discovery-questions.md` for initial and evidence-based question patterns.
- `templates/modular-architecture-plan.md` for planning deliverables.
- `templates/implementation-plan.md` for approved implementation sequencing.
- `templates/adr.md` when the user wants an architecture decision record.
- `templates/code-review.md` for scoped modular/robust architecture reviews.
- `templates/validation-checklist.md` before final handoff.
- `references/modular-architecture-principles.md`, `references/robustness-principles.md`, `references/implementation-playbook.md`, and `references/agent-operating-model.md` when the task needs more detail than this file provides.
- `examples/` for reusable prompt examples, not for task-specific evidence.

## Activation Rules

Use this skill only when the user asks to build, implement, refactor, integrate, modularize, harden, or redesign a concrete target such as:

- feature
- module
- package
- service
- API
- workflow
- data model
- configuration or contract system
- integration
- background job
- automation tool
- repository subsystem
- domain boundary

Do not use this skill for:

- isolated syntax fixes
- small bug fixes with obvious scope and behavior
- formatting or lint cleanup only
- one-off snippets
- package installation help
- generic programming explanations
- broad best-practice reviews with no target
- global rewrites of a whole project unless the user explicitly asks for a whole-project architecture engagement
- applying the same architecture checklist to unrelated files

If the user gives a broad request, narrow it before work begins. Ask which module, subsystem, feature, workflow, integration, or pain point should be handled first.

## Non-Negotiable Operating Rules

1. Ask before work begins. Before scanning, planning, or editing files, ask focused questions about purpose, target, constraints, users, success criteria, risk tolerance, and desired output.
2. Ask useful questions, not generic questionnaires. Infer the first questions from the user's request. Ask only questions that can change the architecture or implementation path.
3. Inspect evidence after the initial answers. Scan the relevant codebase, project, model, schema, config, API, tool, docs, tests, commands, and runtime consumers. Do not rely on assumptions when evidence is available.
4. Ask evidence-based follow-up questions. After scanning, ask additional questions only when the scan reveals uncertainty that would materially change the design.
5. Keep scope narrow. Follow the target lifecycle outward until dependencies are understood. Do not audit unrelated areas just because they exist.
6. Build for modularity. Define boundaries, ownership, public contracts, dependency direction, seams, and extension points before implementation.
7. Build for robustness. Design validation, error handling, recovery behavior, idempotency, security boundaries, observability, migrations, and verification into the implementation.
8. Recommend one path. Provide one decisive architecture and implementation plan by default. Provide alternatives only when the user asks.
9. Implement incrementally after approval. Prefer vertical slices, adapters, compatibility layers, tests, and rollback points over broad rewrites.
10. Verify every step. Every implementation phase must have a test, typecheck, lint, contract check, migration dry run, smoke test, runtime check, or explicit manual verification.
11. Preserve working behavior. Do not break existing public APIs, saved data, deployment flows, or user behavior unless the user explicitly approves the breaking change.
12. State uncertainty. If something was not found or cannot be proven from the project, say so and reduce the plan to safe assumptions.

## Required Workflow

### Phase 1: Purpose and Constraint Interview

Start with three to seven questions before scanning or editing. Choose questions based on the user's prompt.

Question categories:

- Target: What exact feature, module, service, workflow, API, data model, integration, or tool is being built or changed?
- Purpose: What user, operator, or developer outcome should the architecture enable?
- Failure: What is currently fragile, duplicated, slow, unsafe, hard to extend, hard to test, or unclear?
- Constraints: What must remain compatible: public API, database schema, saved data format, events, config shape, deployment flow, permissions, UX, performance, or external contract?
- Consumers: Which layers consume this: UI, API, CLI, background jobs, workers, external providers, data pipelines, automation, tests, operators, or other packages?
- Robustness target: What risks matter most: invalid input, retries, partial failure, concurrency, data integrity, permissions, observability, rollback, scalability, or operational support?
- Output: Should the agent deliver an architecture plan, implementation task list, ADR, code changes after approval, migration plan, or review checklist?

Example first response pattern:

I need to narrow the implementation target before touching the project:

1. What exact module, workflow, API, integration, data model, or system boundary should be changed?
2. What outcome matters most: cleaner modular boundaries, safer implementation, easier extension, stronger validation, better failure handling, or lower migration risk?
3. What existing behavior or contract must not break?
4. Which consumers depend on this path today?
5. Do you want an implementation-ready plan first, or should code changes follow after the plan is approved?

If the user already answered some of these, ask only the missing questions. If the prompt already contains enough detail to proceed, state the interpreted purpose, constraints, and target first, then ask for confirmation or any missing blocker before scanning or editing. Do not skip the initial conversation gate just because the user asked for implementation.

### Phase 2: Targeted Evidence Scan

After the user answers, inspect only the relevant implementation. Start with the target and widen only when the lifecycle requires it.

Scan order:

1. Locate entry points, public interfaces, package exports, routes, commands, jobs, handlers, schemas, migrations, configuration, docs, and tests related to the target.
2. Map the lifecycle: input -> validation -> orchestration -> domain logic -> adapters -> storage or side effects -> output -> observability.
3. Identify module boundaries and ownership: which layer creates, validates, transforms, persists, publishes, renders, observes, retries, or owns each concept.
4. Identify contracts: types, interfaces, schemas, events, API payloads, config formats, database constraints, adapter protocols, and generated outputs.
5. Trace dependency direction. Look for inward dependencies from domain logic to infrastructure, circular imports, hidden globals, and direct cross-module calls.
6. Identify robustness gaps: weak validation, unsafe defaults, missing idempotency, ambiguous error handling, swallowed failures, unbounded retries, race conditions, missing permissions, missing observability, missing rollback, or insufficient tests.
7. Inspect existing patterns that are working well in the repository. Prefer local consistency when it supports modularity and robustness.
8. Inspect tests and commands that can verify the target. Note missing safety harnesses when they affect implementation risk.
9. Check official documentation or source references when framework, SDK, runtime, model, deployment, or tool behavior is uncertain or likely to have changed.

Use lightweight inspection first: `ls`, `find`, `rg`, package manifests, route maps, schema files, migration folders, docs, test lists, and dependency manifests. Avoid expensive or broad scans unless needed.

### Phase 3: Evidence-Based Follow-Up

After scanning, ask follow-up questions only when the answer changes architecture or implementation.

Ask when evidence reveals:

- competing implementations for the same responsibility
- unclear authoritative source of truth
- a public contract that may need compatibility support
- saved data or migrations that can break existing users
- hidden consumers outside the scanned path
- ambiguous ownership between config, domain logic, adapters, and storage
- missing tests around risky behavior
- external provider constraints such as authentication, rate limits, retries, webhooks, timeouts, idempotency, or audit logs
- generated files that may or may not be safe to edit
- conflicting domain vocabulary or duplicated concepts
- a seam that can be used for incremental migration

Do not ask follow-up questions to delay action. If the answer is unlikely to change the plan, proceed with an explicit assumption.

### Phase 4: Modular Architecture Design

Design the smallest architecture that solves the target problem and creates clear seams for future change.

Required modularity checks:

- Name the module or subsystem boundary.
- Define what the module owns and what it must not own.
- Define public contracts: interfaces, DTOs, events, schemas, config, package exports, or API routes.
- Define private internals that should not be consumed directly.
- Define dependency direction. Domain and core logic should not depend directly on UI, framework glue, provider SDKs, databases, global state, or generated artifacts unless the project already has a deliberate pattern.
- Define adapters for external systems, providers, storage, queues, file systems, CLI, UI, and legacy code.
- Define composition root or wiring location where concrete implementations are connected.
- Define source of truth for domain vocabulary, configuration, contracts, and generated output.
- Define migration or compatibility seam when replacing behavior.

Prefer these patterns when evidence supports them:

- ports and adapters around external systems
- thin framework entry points that call application services
- domain services or use-case handlers for business workflows
- repositories or gateways for persistence boundaries when the project already uses them
- anti-corruption layers for legacy or third-party formats
- contract-first schemas for public APIs, events, config, and generated artifacts
- normalized internal representations for config-heavy or manifest-heavy systems
- feature flags or compatibility adapters for risky migrations
- package exports that expose stable APIs and hide internals

Avoid speculative abstractions. Do not introduce a plugin framework, event bus, service mesh, queue, generator, schema system, dependency injection container, or microservice boundary unless the inspected evidence and user goal justify it.

### Phase 5: Robustness Design

Design robustness where it belongs. Do not bolt it on globally.

Required robustness checks for the target:

- Validation: reject invalid input early; reject unknown keys where strictness is required; validate references, duplicate definitions, incompatible combinations, and unsafe defaults.
- Errors: define typed or structured errors where the project supports them; preserve useful context; avoid swallowing failures; define user-safe and operator-safe messages.
- Idempotency: required for retries, webhooks, background jobs, external writes, imports, payments, provisioning, emails, and deployment automation.
- Retries and timeouts: bounded, observable, and applied only to operations that are safe to retry.
- Transactions and consistency: define atomic sections, rollback behavior, compensation, or eventual consistency where relevant.
- Concurrency: consider locking, deduplication, optimistic concurrency, race-prone state transitions, and queue semantics.
- Permissions and trust boundaries: authorize before side effects; keep security decisions in code with tests, not scattered config.
- Observability: logs, metrics, traces, audit records, correlation IDs, health checks, and actionable failure states where relevant.
- Performance: define expected scale, hot paths, caching rules, and backpressure only when evidence shows a constraint.
- Migration: include compatibility layers, data migrations, parity checks, generated output diffs, rollback plans, and stop conditions.
- Tests: unit, integration, contract, schema, migration, snapshot, property, smoke, or end-to-end tests selected for the target.

### Phase 6: Implementation Plan

Before changing files, produce an implementation-ready plan unless the user explicitly requested direct implementation after the interview.

The plan must include:

1. Target scope and non-scope.
2. Evidence inspected: files, symbols, commands, schemas, tests, docs, or runtime paths.
3. Current lifecycle and module map.
4. Current weaknesses tied to evidence, not generic claims.
5. Target modular architecture.
6. Robustness design for validation, failures, migration, and verification.
7. Ordered implementation phases.
8. Files to change first and files to avoid.
9. Verification command or check for every phase.
10. Stop conditions and rollback points.
11. Acceptance criteria.

### Phase 7: Implementation Execution

When the user approves implementation, execute the plan in small, verifiable slices. If the user requested implementation from the start, still complete the initial conversation gate, targeted scan, and implementation plan before editing files.

Implementation rules:

- Start with tests, contracts, schema checks, or safety harnesses when current risk is high.
- Create or strengthen public contracts before changing internals.
- Add the seam before moving behavior.
- Keep legacy compatibility until parity is proven.
- Implement one vertical slice first when practical.
- Update docs only for changed behavior or public contracts.
- Remove transitional code only after verification.
- Do not opportunistically refactor unrelated code.
- Do not introduce dependencies unless the project lacks a reasonable built-in option and the benefit is explicit.
- Run the most relevant available checks.
- If checks cannot run, explain why and provide the exact command the user should run.

### Phase 8: Final Response

When planning only, return:

- scope
- evidence scanned
- current architecture map
- target architecture
- implementation plan
- robustness plan
- verification plan
- stop conditions
- acceptance criteria
- unresolved questions, only if they genuinely affect implementation

When code was changed, return:

- what changed
- why the modular boundary is better
- what robustness was added or preserved
- files changed
- tests or checks run
- checks not run and why
- migration or rollout notes
- remaining risks

## Architecture Decision Rules

### Boundaries before abstractions

A module is not modular because it has more folders. It is modular when ownership, public contracts, private internals, dependency direction, and verification points are clear.

### Contracts before cleverness

A robust system has explicit contracts for the parts that cross boundaries: API payloads, events, config, database constraints, file formats, provider requests, package exports, CLI inputs, and generated artifacts.

### Core before adapters

Keep business rules and system policy in the core or application layer. Keep framework glue, provider SDKs, persistence, filesystem access, UI rendering, queues, and external formats behind adapters when the target requires a seam.

### Source of truth must be named

The source of truth may be a domain model, schema, database table, generated artifact, config file, event contract, API route, package export, or service. It should be named, owned, versioned when needed, and consumed consistently.

### Config is intent, code is behavior

Configuration should express stable intent, values, capabilities, mappings, and constraints. Runtime branching, security rules, orchestration, side effects, and complex behavior belong in code with tests.

### Normalize before consuming

For config, manifest, schema, generated-contract, or import systems, prefer this lifecycle when relevant:

compact source -> strict validation -> reference or preset resolution -> normalized internal representation -> runtime consumers

The normalized representation can be verbose when it is deterministic, generated, versioned, and testable.

### Robustness is scoped

Do not add retries, queues, circuit breakers, caching, distributed locks, service extraction, or observability everywhere. Add the robustness mechanism where the failure mode exists and where a verification check can prove it.

### Migration is architecture

A safe migration is part of the design. Use compatibility layers, adapters, feature flags, backfills, dual reads or writes, parity tests, generated diffs, dry runs, and stop conditions when the target affects existing users or data.

### Tests prove the boundary

Tests should prove public contracts, validation, behavior, failure modes, migrations, and adapter behavior. Do not rely only on snapshots or happy-path tests for robust systems.

### Small steps beat grand rewrites

A correct modular architecture can often be introduced by a seam, a contract, a validation layer, and a vertical slice. Avoid project-wide rewrites unless the user explicitly chooses that risk.

## Agent Safety Rules

- Do not fabricate files, APIs, models, schemas, tests, or commands.
- Do not make global claims from a narrow scan.
- Do not change files before the user has had a chance to answer the initial questions unless the user explicitly requested direct implementation.
- Do not refactor unrelated code during implementation.
- Do not move responsibilities across module boundaries without preserving public behavior or documenting the breaking change.
- Do not convert everything into configuration.
- Do not convert every seam into a framework.
- Do not add resilience mechanisms that make failure harder to understand.
- Do not ignore existing repository conventions unless they conflict with the stated goal or observed risk.
- Do not ask questions that can be answered by scanning the project.

## Output Quality Bar

A useful result should be specific enough that another agent can implement it without repeating the analysis. It should name concrete files, functions, classes, packages, schemas, commands, and tests when available. It should separate observed evidence from assumptions. It should include verification and stop conditions. It should improve modularity and robustness for the target without spreading generic rules across unrelated code.

---
> Source: [savedpixel/modular-robust-systems-architect](https://github.com/savedpixel/modular-robust-systems-architect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
