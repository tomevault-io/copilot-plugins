## dynastyiq

> These rules govern all Codex-assisted development on this repository.

# AGENTS.md — Codex Repository Rules

These rules govern all Codex-assisted development on this repository.
This file exists to bootstrap new LLM sessions and enforce project invariants.

---

## Authoritative Inputs (Must Be Read First)

Before proposing a plan or writing any files, Codex MUST read and treat the following as authoritative.

### Core Context

- README.md
- docs/summary.md
- docs/CONVENTIONS.md
- docs/ENUMS.md
- docs/UI_DESIGN.md
- docs/UI_MOTION.md
- docs/testing/testing-standards.yaml
- docs/architecture/README.yaml
- docs/ARCHITECTURE_INVENTORY.md
- docs/pr-plans/current_pr.md, when present

### Architecture (Primary Source of Truth)

- docs/architecture/README.yaml
- docs/architecture/**/*.yaml

These YAML files define canonical domain rules, invariants, and approved abstractions.

### Testing (Primary Source of Truth for Test Work)

- docs/testing/testing-standards.yaml
- docs/architecture/testing/*.yaml

These files define automated test completeness, sufficiency, deterministic behavior, and approved testing patterns.

### Architecture Inventory (Derived / Bootstrap Only)

- docs/ARCHITECTURE_INVENTORY.md

This file exists solely to help bootstrap new LLM chats.
It is **not** the primary source of architectural truth.

### Database (Order of Authority)

1. database/migrations/** (source of truth)
2. docs/DB_SCHEMA.md (contextual reference only)

### UI (Order of Authority)

1. docs/CONVENTIONS.md
2. docs/UI_DESIGN.md
3. docs/UI_MOTION.md
4. docs/ui_backlog.md (known existing deviations only)
5. resources/views/**
6. resources/js/**

Existing deviations in `docs/ui_backlog.md` do not permit new deviations.

### Dependency Reality

- composer.lock
- package-lock.json
- diq-bot/package-lock.json

If conflicts are detected between authoritative sources, work MUST pause and be escalated to the human.

### Active PR Plan

- `docs/pr-plans/current_pr.md` is the active working PR plan when present.
- Codex MUST read it before proposing a plan or editing files.
- PR-plan aliases, metadata, ID assignment, archiving, and promotion are governed by `docs/pr-plans/pr-workflow.yaml`.
- The current PR plan is working context, not permanent architecture authority.
- Durable decisions from the current PR plan MUST be promoted to the canonical docs they affect, such as `docs/architecture/**/*.yaml`, `docs/ENUMS.md`, `docs/DB_SCHEMA.md`, or `docs/CONVENTIONS.md`.
- Candidate future PR plans live under `docs/pr-plans/backlog/`.
- Completed, closed, or abandoned PR plans should be snapshotted under `docs/pr-plans/archive/`.

---

## Workflow

- Codex must never proceed based on inferred or partial intent.
- Discussion, diagnosis, clarification, or design conversation is not approval to modify code, tests, docs, configuration, or other repository files.
- Bug reports, observations, screenshots, examples of incorrect behavior, or statements that something is wrong are not approval to modify files.
- Codex must always propose a concrete plan before implementing anything or changing code, tests, docs, configuration, or other repository files.
- Codex must not propose an implementation plan until it is greater than 95% certain of the human's requirements; if certainty is lower, Codex must ask clarifying questions first.
- After proposing a plan, Codex must wait for explicit human approval before modifying files.
- Codex may only edit files after the human explicitly asks for implementation or file changes, using clear wording such as “implement,” “make the change,” “fix it,” “update the file,” “create,” or equivalent.
- When the human appears to be discussing options or asking conceptual questions, Codex must respond in discussion mode only and must not run write operations.
- When editing is approved, Codex must keep changes scoped to the requested outcome.

---

## CI & Execution Policy (Strict)

- Codex MUST NOT run `./ci.sh` or any test/CI commands unless explicitly instructed.
- Codex MUST NOT run import, sync, queue, scheduler, bot, migration, seed, or destructive data commands unless explicitly instructed.
- The human is responsible for running all CI, tests, scripts, imports, and operational commands.
- Codex MAY:
    - Create and modify test files
    - Propose improvements to tests
    - Propose the exact commands the human should run
- Codex must stop after writing code/tests/docs and await human review.

---

## Completion Gate (Non-Negotiable)

Codex may NOT declare a task, PR, or change set “complete”, “finished”, or “ready”
until the human explicitly approves completion in chat.

Codex output MUST end in one of:

- “Awaiting human review”
- “Awaiting approval to proceed”
- “Awaiting requested changes”

Codex must never self-certify completion.

---

## Change Discipline

- Prefer the smallest possible change.
- Never refactor unless explicitly requested.
- Never introduce new top-level directories without approval.
- Do not modify dependencies without explicit approval.
- Do not modify migrations after they have been applied.
- Do not introduce global JavaScript state unless explicitly approved.
- Preserve existing route names, authorization checks, and public interfaces unless the human explicitly approves a breaking change.

---

## Standards

- PHP code must follow PSR-12.
- PHPDoc is required per `docs/CONVENTIONS.md`.
- New automated PHP tests must use Pest.
- Test authoring and test audits must follow `docs/testing/testing-standards.yaml`.
- New UI and materially touched UI must follow `docs/UI_DESIGN.md`.
- Interactive state changes must follow `docs/UI_MOTION.md`.
- Existing architectural patterns and invariants must be respected.
- Enum-like values must be documented in `docs/ENUMS.md`.
- Database schema documentation must stay descriptive; migrations remain the source of truth.

---

## Certainty & Communication

- Never act without high certainty of requirements.
- Codex must be greater than 95% certain of the human's requirements before proposing an implementation plan.
- If requirement certainty is not greater than 95%, Codex must ask one clarifying question at a time before proposing a plan.
- Ask clarifying questions one at a time.
- Always state current certainty level before asking a question.
- Do not infer intent from partial context.
- If a requested change conflicts with authoritative docs, stop and ask.
- If code and docs disagree, stop and ask unless the task explicitly asks to resolve the disagreement.

---

## Test-Driven Pull Requests

- All PRs are test-driven by default.
- Codex may write initial or scaffolded tests.
- Tests are expected to be refined collaboratively with the human.
- Test work must follow `docs/testing/testing-standards.yaml`.
- For test-authoring or test-audit tasks, Codex must complete the acknowledgment gate defined in `docs/testing/testing-standards.yaml` before editing test files.
- For test-authoring or test-audit tasks, Codex must provide the coverage matrix, requirements checklist, auth/scope matrix where applicable, frontend component checklist where applicable, and per-file test count report required by `docs/testing/testing-standards.yaml`.

### UI and Content Changes

- Pure copy/layout changes do not require tests if no behavior is affected.
- Changes affecting legal, financial, billing, membership, imports, integrations, authorization, or operational content require lightweight test coverage unless the human explicitly defers tests.
- Existing tests must be updated if they cover modified behavior.

---

## Creating New Abstractions

- Default to existing abstractions defined in `docs/architecture/**`.
- If a new abstraction is proposed, Codex MUST:
    - State the problem it solves
    - Explain why existing architecture YAML files are insufficient
    - Propose the minimal surface area
    - Identify the correct `docs/architecture/<domain>/` location

### Approval & Documentation Flow

- New abstractions require explicit human approval.
- Once approved:
    - A new YAML file MUST be added under `docs/architecture/**`
    - The file MUST follow the schema defined in `docs/architecture/README.yaml`
    - The abstraction MUST enforce invariants, not implementation trivia
- Afterward, `docs/ARCHITECTURE_INVENTORY.md` MUST be updated only as a derived summary for future LLM bootstrapping.

---

## Codebase-Specific Rules

### Laravel App

- Laravel route, controller, model, policy, job, and service changes must respect `docs/architecture/application/LaravelApplicationShell.yaml`.
- Admin import changes must respect:
    - `docs/architecture/admin/AdminImportRegistry.yaml`
    - `docs/architecture/admin/ImportBroadcastStream.yaml`
- NHL import changes must respect:
    - `docs/architecture/imports/NhlImportOrchestrator.yaml`
    - `docs/architecture/imports/NhlDiscoveryPipeline.yaml`
    - `docs/architecture/imports/NhlGameDataImportServices.yaml`
    - `docs/architecture/imports/ImportProgressRepository.yaml`

### Integrations

- Fantrax changes must respect:
    - `docs/architecture/integrations/FantraxUserConnection.yaml`
    - `docs/architecture/integrations/FantraxLeagueSync.yaml`
    - `docs/architecture/integrations/PlatformStateService.yaml`
- Patreon changes must respect `docs/architecture/integrations/PatreonProviderSync.yaml`.
- Discord integration changes must respect:
    - `docs/architecture/integrations/DiscordServerConnection.yaml`
    - `docs/architecture/integrations/DiscordBotBridge.yaml`
    - `docs/architecture/application/DiscordBotRuntime.yaml`

### UI

- UI authority is documented in `docs/architecture/ui/UIDesignAuthority.yaml`.
- UI motion standards are documented in `docs/UI_MOTION.md`.
- Existing deviations are tracked in `docs/ui_backlog.md`.
- New UI deviations are prohibited unless explicitly approved.
- Blade executable `<script>` blocks must not be introduced into new or migrated interactive pages.
- Page-local JavaScript must not rely on global application state.

---

## Prohibited Actions (Unless Explicitly Approved)

- Making changes directly on the default branch
- Auto-committing or auto-merging
- Refactoring beyond the approved scope
- Introducing global state or hidden side effects
- Modifying architecture, dependencies, conventions, or schemas beyond the requested scope
- Running CI, tests, scripts, imports, bots, queues, schedulers, migrations, or seeders
- Proceeding with unclear requirements
- Reading, indexing, or modifying ignored paths

---

## File Update Discipline

- Prefer targeted edits for small changes.
- When changing core logic, Codex should prefer coherent whole-file edits over disconnected snippets, unless that would increase risk.
- Documentation changes must keep authority boundaries clear and avoid duplicating enum values, schema definitions, or UI rules outside their canonical files.

---

## Don't Modify
The agent must never modify files matching these patterns:
- *.log
- .env
- .env.*


## Ignore Paths
The agent must NEVER read, index, modify, or reason about files matching these patterns:

- storage/**
- vendor/**
- node_modules/**
- public/build/**
- bootstrap/cache/**

---
> Source: [unohuim/dynastyiq](https://github.com/unohuim/dynastyiq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
