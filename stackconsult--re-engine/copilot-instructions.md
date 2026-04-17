## re-engine

> - Rule 1 defines how we use Qwen3‑Coder‑Next via Ollama as our primary coding agent.


# Global Agent-Friendly Repo & Coding Conventions

## Context

- Rule 1 defines how we use Qwen3‑Coder‑Next via Ollama as our primary coding agent.
- This rule defines how repositories and code should be structured so agents (Qwen + Cascade) can work reliably, repeatedly, and at production quality.
- Humans and agents share the same repos; structure and conventions must serve both.

## 1. Agent-first repository layout

All production repos should converge on an agent-friendly layout:

### Root structure

- `src/` → application code
- `tests/` → tests (unit, integration, E2E subfolders as needed)
- `docs/` → documentation and design
  - `docs/dev-setup.md` → dev environment and dependency setup
  - `docs/OPERATIONS.md` → runbooks and operational tasks
  - `docs/plans/` → design and refactor/bugfix plans
  - `docs/decisions/` → architecture/decision records (ADR-style)

### Agent guide

- Add `AGENTS.md` at the repo root that explains:
  - What this repo does.
  - Where main entrypoints live (CLI, HTTP, jobs).
  - How to run dev, tests, and key workflows.
  - Where to find and how to update design docs, plans, and decisions.
- Keep structure consistent across repos so agents can reuse patterns.
  - Similar problem domains should have similar layouts and naming.

## 2. Coding stack defaults (can be overridden per repo)

Unless a repo explicitly defines alternatives in project-specific rules:

### Backend services

- **Primary:** TypeScript (Node.js) with an HTTP framework such as Express, Fastify, or Nest.
- **Secondary:** Python (FastAPI) when Python ecosystems or ML tooling are required.

### Frontend / UI

- React + TypeScript as the default when a SPA is needed.
- Tailwind CSS preferred for styling with React.

### Automation / jobs / workers

- TypeScript (Node.js) or Python, aligned with the main backend.
- Use queues (e.g., Redis/BullMQ, SQS, etc.) for long-running or retryable work.

### Infrastructure-as-code

- Terraform or Pulumi where cloud infra is managed as code.
- Docker as the default packaging format, with one primary Dockerfile per service.

### Agents (including Qwen3‑Coder‑Next) MUST

- Prefer these stacks when starting new services or modules, unless project rules say otherwise.
- Extend existing stack choices; do not introduce a second HTTP framework, ORM, or queue library if one is already established in the repo.

## 3. Coding style & testing expectations

### General style

- Prefer small, composable functions and modules.
- Avoid global mutable state; rely on explicit wiring / dependency injection.
- Always add explicit error handling; never swallow errors silently.
- Align with existing patterns in the repo before introducing new ones.

### TypeScript

- Use `strict: true` in tsconfig for new projects.
- Prefer ES modules and async/await.
- Keep modules focused and cohesive; avoid "god objects" and mega files.

### Python

- Target Python 3.10+ unless specified otherwise.
- Use type hints for all new code; align with mypy or equivalent when configured.
- Separate IO (HTTP, DB, file system) from pure business logic where practical.

### Testing

- **Every new feature:**
  - At least one happy-path test and one failure/edge-case test.
  - Tests must be runnable via the primary test command documented in `docs/dev-setup.md` or `docs/OPERATIONS.md`.
- **Every bugfix:**
  - Add or update a test that fails before the fix and passes after.
  - No fix is complete without test coverage.
- **Test placement:**
  - Unit tests close to code (e.g., `src/**/__tests__` or `tests/unit`).
  - Integration/E2E tests for critical workflows (e.g., job queues, external APIs, main business flows).

## 4. Documentation & knowledge organization for agents

Docs are critical for reliable agentic behavior:

### dev-setup

- `docs/dev-setup.md` must describe:
  - Required OS and tools (including Ollama and Qwen model requirements).
  - Steps to set up language runtimes and build tools.
  - Exact commands for install, dev, test, lint, format.

### operations

- `docs/OPERATIONS.md` must include:
  - How to run main operational tasks (batch jobs, imports, maintenance).
  - How to run health checks and smoke tests.
  - Any manual steps needed before/after deployments.

### plans

- `docs/plans/` stores:
  - Feature designs.
  - Refactor plans.
  - Bugfix plans (e.g., `bug-<id>.md`) created/updated as part of Qwen workflows.

### decisions

- `docs/decisions/` (ADR-style) records:
  - Long-lived architecture choices.
  - Tradeoffs and why certain stacks/libraries are preferred.

### Agents MUST read these docs before making large or risky changes, and update them when introducing new patterns, workflows, or decisions

## 5. CI, workflows, and skills alignment

Repos must expose a minimal, consistent interface for Cascade skills and workflows:

### Commands

- Standardize on a Makefile or equivalent package scripts that define:
  - `install`
  - `dev`
  - `test`
  - `lint`
  - `format`
- `docs/dev-setup.md` and/or `AGENTS.md` must point to these commands.

### Workflows

- Skills like `setup-dev-environment`, `design-feature`, `implement-feature`, `refactor-with-tests`, `secure-and-harden`, `release-to-env`, and `qwen-bugfix-loop` should be able to:
  - Discover and run the right commands without guessing.
  - Log their effects in `docs/plans/` and `docs/CHANGELOG.md` (or equivalent).

### CI

- Local and remote CI should run the same core commands (`test`, `lint`, `build`).
- Agents should treat local test runs as authoritative when using Qwen in loops.

## 6. How agents should behave with these conventions

### Agents (Qwen3‑Coder‑Next via Ollama, other models, and Cascade itself) MUST

- Prefer and reinforce this repo layout and command structure.
- Reuse existing patterns in the repo instead of inventing new ones.
- Keep changes small, reviewable, and test-backed, aligned with Rule 1's agentic loops.

### When conventions are missing

- Propose concrete changes (new folders, Makefile, docs) to align the repo with this rule.
- Ask for explicit approval before large structural changes.
- Document what was done in `docs/dev-setup.md`, `AGENTS.md`, and the relevant plan/decision docs.

## Summary

- Rule 1 defines how we drive Qwen via Ollama.
- This rule defines how code and repos must look so those agents can operate as a reliable, repeatable production factory.
- Agents should converge repos toward this standard over time while respecting project-specific overrides and constraints.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stackconsult) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
