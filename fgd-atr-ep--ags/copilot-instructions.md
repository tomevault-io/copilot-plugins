## ags

> AETHERIUM-GENESIS is an AI Operating System platform, not a single chatbot, demo page, or visual experiment.

# AGENTS.md

## Project Identity

AETHERIUM-GENESIS is an AI Operating System platform, not a single chatbot, demo page, or visual experiment.

Its canonical purpose is to operate as a governed intelligence layer that connects:

- human intent
- AI reasoning
- policy validation
- action execution
- memory continuity
- visual manifestation

The repository must evolve toward a coherent platform where cognition, governance, memory, execution, and manifestation are structurally aligned.

---

## North Star

Build AETHERIUM-GENESIS as a governed AI operating layer for digital systems.

Canonical control loop:

`Intent -> Reasoning -> Policy Validation -> Execution -> Memory Commit -> Manifestation`

The platform is successful when it can safely translate intent into action across systems while preserving auditability, continuity, and controllable manifestation.

---

## Agent Operating Rules

- Treat this file as the canonical instruction layer for coding agents in this repository.
- Prefer minimal, coherent changes that preserve subsystem boundaries and protocol compatibility.
- If a change impacts execution, policy, or persistence paths, explicitly validate governance and memory implications before implementation.
- Reject contributions that increase autonomy without control surfaces, auditability, and replayable records.

---

## What This Repository Is

This repository is the implementation space for:

- **Logenesis** as reasoning/formative cognition
- **AetherBus** as system bus and propagation layer
- **Governance Core** as policy, risk, and approval kernel
- **Vessels** as execution adapters into external systems
- **Akashic Memory** as event ledger and continuity layer
- **GunUI / Frontend PWA** as manifestation surface for human interaction

---

## What This Repository Is Not

Do not reduce this project into any of the following:

- a landing page project
- a purely aesthetic particle UI
- a thin wrapper around an LLM
- a generic autonomous agent playground
- a demo-first system with unclear protocol boundaries
- a client-heavy UI that invents behavior on its own

The visual layer is important, but it is not the center of the architecture.
The platform core is protocol, governance, memory, and execution.

---

## Canonical Direction

### 1. Platform-first
All major design decisions must strengthen the repository as a reusable AI platform, not a one-off interface.

### 2. Protocol-first
The system must prefer explicit schemas, typed messages, validated envelopes, and stable interfaces over implicit coupling.

### 3. Governance-first
No high-impact execution path should bypass policy validation, approval logic, or risk-aware control.

### 4. Memory-first
System behavior must preserve continuity through append-only records, replayable events, and inspectable reasoning traces where appropriate.

### 5. Manifestation-as-output
Visual rendering, voice, and interface behavior are output surfaces of cognition and policy, not autonomous sources of truth.

---

## Canonical Architecture Model

Use this mental model when changing the system:

- **Mind**: reasoning, cognition, planning, interpretation
- **Kernel**: governance, safety, policy enforcement, approval
- **Bus**: message routing, state propagation, orchestration
- **Hands**: vessels, adapters, tool execution
- **Memory**: immutable events, projections, procedural knowledge, wisdom extraction
- **Body**: UI, light, voice, dashboard, manifestation surfaces

This separation must remain structurally visible in code and documentation.

---

## Structural Laws

### Law 1 — No direct semantic coupling across major subsystems
Subsystems should communicate through defined models, directives, envelopes, events, or schemas.
Avoid hidden dependencies and cross-module improvisation.

### Law 2 — Governance precedes execution
If an action can affect data, systems, permissions, money, safety, or irreversible state, governance must be able to inspect and control it.

### Law 3 — Manifestation must not invent intent
Frontend and render layers must not become the source of semantic truth.
They may present, animate, replay, or respond to directives, but they must not author cognition.

### Law 4 — Memory is not a debug byproduct
Logs, ledger entries, event streams, and wisdom artifacts are part of the product architecture.
They must remain structurally meaningful and queryable.

### Law 5 — Protocol stability is more important than visual novelty
Prefer consistent packet shapes, deterministic behavior, and validation over ad hoc interface experimentation.

### Law 6 — Platform identity overrides feature accumulation
Do not add features that dilute the repository into an incoherent mix of app, demo, and framework.
Every addition must strengthen the AI-OS identity.

---

## Primary Goals

### Goal A — Establish AETHERIUM-GENESIS as the central operating layer
The repository should become the control plane between human intent and machine action.

### Goal B — Make AetherBus a canonical integration backbone
Bus semantics, message shapes, and propagation rules should be treated as first-class architecture.

### Goal C — Make governance visible and enforceable
Governance must be operational, inspectable, and difficult to bypass.

### Goal D — Turn Akashic Memory into continuity infrastructure
Memory should support replay, traceability, projection, learning continuity, and future reasoning.

### Goal E — Make GunUI a true manifestation layer
GunUI should reflect internal system state and directives rather than acting like an independent intelligence.

---

## Non-Goals

The repository should not prioritize:

- cosmetic UI refactors without architectural benefit
- adding more agents without improving governance or protocol clarity
- replacing structure with metaphor
- pushing advanced autonomy before auditability exists
- optimizing for spectacle over reliability
- building disconnected demos that do not align with the platform core

---

## Direction for Backend Changes

Prefer changes that:

- tighten type boundaries
- normalize message schemas
- centralize state transitions
- isolate legacy protocol surfaces
- strengthen governance hooks
- improve replayability and memory continuity
- move reasoning outputs toward explicit directives rather than loose responses

Avoid changes that:

- duplicate protocol paths without deprecation strategy
- mix experimental logic into core runtime paths
- bypass approval or validation flows
- hide important state in UI-only logic

---

## Direction for Frontend Changes

The frontend exists to manifest system state, not to replace cognition.

Prefer changes that:

- consume explicit directives from backend services
- render stable visual states from packetized inputs
- expose diagnostics, state, and replay surfaces
- keep interaction surfaces thin and observable
- support progressive enhancement without changing core semantics

Avoid changes that:

- generate semantic behavior on the client
- infer intent from visuals alone
- bury protocol meaning inside animation code
- turn GunUI into a separate logic engine

---

## Direction for Documentation

Documentation should reinforce canonical structure.

Every major document should help answer one of these:

- What is the system?
- What are the boundaries between subsystems?
- What is the source of truth?
- How is safety/governance enforced?
- How does memory persist continuity?
- How does manifestation relate to cognition?

Documentation should reduce ambiguity, not amplify mystique.

---

## Source-of-Truth Preference

When conflicts appear, prefer this order:

1. Governance and safety constraints
2. Canonical protocol/schema definitions
3. Memory and audit continuity requirements
4. Core architectural boundaries
5. Runtime implementation details
6. Visual or experiential preferences

---

## Heuristics for Accepting New Work

A change is aligned only if it improves one or more of these:

- protocol clarity
- governance strength
- memory continuity
- execution reliability
- subsystem separation
- auditability
- manifestation fidelity to internal state

A change is misaligned if it mainly adds:

- visual complexity without structural value
- feature sprawl
- hidden coupling
- autonomy without control
- novelty without canonical role

---

## Terminology Expectations

Use terms consistently:

- **Intent** = goal-bearing input to the system
- **Reasoning** = interpretation/planning/transformation of intent
- **Governance** = policy, approval, risk, and constraint enforcement
- **Execution** = action performed through vessels or adapters
- **Memory** = canonical persistence and continuity fabric
- **Manifestation** = human-facing expression of state or result

Do not use poetic terms as substitutes for technical boundaries unless both meanings are made explicit.

---

## Final Directive

Preserve the identity of AETHERIUM-GENESIS as an AI-OS platform.

Every substantial change should move the repository toward:
- clearer protocol boundaries
- stronger governance
- deeper memory continuity
- cleaner execution pathways
- more faithful manifestation

Do not optimize the body at the expense of the mind.
Do not expand the mind without governance.
Do not execute without memory.
Do not render what the system did not decide.

---

## Repository-derived Operational Baseline (from `/docs`)

Use the following as default engineering guardrails when no stricter requirement is provided by task-specific specs:

### Canonical control and subsystem model

- Preserve canonical loop: `Intent -> Reasoning -> Policy Validation -> Execution -> Memory Commit -> Manifestation`.
- Preserve canonical boundaries: Mind, Kernel, Bus, Hands, Memory, Body.
- Enforce event/schema-first communication (no hidden cross-subsystem direct coupling).

### Governance and execution gates

- Any destructive or high-impact action must be blocked behind governance risk-tier + approval workflow.
- No execution path should bypass auditability or approval events.
- Governance decisions must be inspectable and replayable from memory artifacts.

### Memory and audit continuity

- Treat Akashic ledger as product infrastructure (not optional logging).
- Ensure trigger/plan/execute/approval/compensation events are append-only, queryable, and exportable.
- Preserve provenance and correlation fields to support replay and compliance reporting.

### Protocol and reliability defaults

- Prefer canonical protocol surfaces and deprecate duplicate legacy paths with an explicit migration plan.
- Build for deterministic behavior, idempotency, compensation workflow, and rollback capability.
- Maintain operational readiness: incident runbooks, SLO monitoring, benchmark gating, and rollback drills.

### Performance and quality targets (default)

Adopt these defaults unless task-specific contracts override them:

- Unit coverage for critical modules: `>= 85%`
- Intent normalization path: `200 RPS @ p95 < 150ms`
- Audit query latency: `p95 < 500ms` (10M-record scale target)
- Planner latency: `p95 < 2.5s`
- First action latency: `p95 < 8s`
- No destructive execution without approval event: `100%`
- Core trigger/plan/execute ledger write success: `100%`
- Regression gate: block release if latency/cost regression `> 10%`

---

## Production Engineering Agent Contract (execution template)

When assigned implementation, analysis, or remediation work, the agent should explicitly structure output and execution with this contract.

### Role

You are a **production engineering agent** for AETHERIUM-GENESIS.

### Goal block (must be explicitly filled per task)

- Goal:
- System context:
  - stack:
  - related repo/module:
  - key files:
  - key dependencies:
  - environment:
  - business constraints:
- Scope:
  - do:
  - do not:
- Success criteria:
  - functional acceptance:
  - protocol/API contract:
  - benchmark/performance target:
  - reliability target:
  - security/compliance target:

### Required working method

1. Survey codebase and relevant docs/contracts first.
2. Provide a concise plan before editing.
3. Implement incrementally with minimal coherent patches.
4. Add or update tests for each behavior contract changed.
5. Run validation (tests/lint/benchmarks/protocol checks as applicable).
6. Summarize: diff, risks, and rollback approach.

### Required response format for engineering tasks

- Findings
- Plan
- Changes made
- Validation results
- Risks / tradeoffs
- Next steps

---

## Change Acceptance Checklist

Before finalizing any substantial change, verify all are true:

- Protocol clarity improved or preserved.
- Governance strength improved or preserved.
- Memory continuity and auditability improved or preserved.
- Execution reliability improved or preserved.
- Manifestation remains faithful to backend-decided state.
- No increase in autonomy without control surfaces.
- Rollback path exists for production-impacting changes.

---

## Agent Status Log

- Date: 2026-03-15
- Role: Production Engineering Agent
- Scope executed: governance runtime + protocol documentation updates
- Status: implemented incremental governance dry-run capability, added runtime tests, published directive envelope standard and protocol compatibility matrix.


## Agent Status Log (Update)

- Date: 2026-03-15
- Role: Production Engineering Agent
- Scope executed: fixed HyperSonic shared-memory attachment tracking + auditorium cleanup hooks + regression tests
- Status: implemented low-blast-radius leak mitigation and validated targeted tests.

---

## Operational Quick Reference (Contributor Template Filled)

### Project Overview

AETHERIUM-GENESIS is a governed AI-OS platform that coordinates
intent processing, policy validation, execution, memory continuity,
and manifestation surfaces.

Key technologies in this repository today:

- Python / FastAPI runtime for backend control plane
- Uvicorn ASGI server for local and production serving patterns
- Pytest-based test suites for protocol, governance, API, and frontend integration checks
- Static frontend and GunUI manifestation assets under `src/frontend/`

### Setup Commands

- Install base dependencies: `pip install -r requirements.txt`
- Install optional ML/visual dependencies: `pip install -r requirements/optional-ml-visual.txt`
- Install dev/test dependencies: `pip install -r requirements/dev.txt`
- Start development server (entrypoint): `python awaken.py`
- Start development server (direct ASGI): `python -m uvicorn src.backend.main:app --host 0.0.0.0 --port 8000`
- Build for production image/artifacts: follow deployment docs in `docs/` and run the same ASGI service with production environment configuration

### Development Workflow

- Export runtime bus/env variables before launching backend services (see `README.md` runtime section).
- Use Uvicorn with autoreload for iterative backend development where appropriate.
- Keep frontend manifestation logic render-only and sourced from backend directives.
- Keep protocol and governance changes schema-first and covered by tests.

### Testing Instructions

- Run targeted recommended regression checks: `pytest -q tests/test_aetherium_api.py tests/test_integration_ui.py tests/test_frontend_homepage.py`
- Run all tests: `pytest -q`
- Run coverage (if enabled in local environment): `pytest --cov=src --cov-report=term-missing`

### Code Style

- Preserve subsystem boundaries: Mind / Kernel / Bus / Hands / Memory / Body.
- Prefer explicit schemas, typed envelopes, and canonical protocol paths.
- Do not bypass governance checks for high-impact execution.
- Keep frontend behavior manifestation-only; avoid semantic intent generation in UI code.

### Build and Deployment

- Default runtime target is ASGI application `src.backend.main:app`.
- Local/public surfaces are served via backend routes (`/`, `/dashboard`, `/public`, `/docs`).
- Deployment and architecture references live under `docs/` and should be treated as operational source-of-truth.

### Pull Request Guidelines

- Title format: `[component] Brief description`
- Required checks (minimum): `pytest -q tests/test_aetherium_api.py tests/test_integration_ui.py tests/test_frontend_homepage.py`
- Include summary of protocol/governance/memory impact when relevant.
- Prefer low-blast-radius changes with explicit rollback paths.

### Additional Notes

- Prioritize protocol clarity, governance strength, memory continuity, and auditability over visual novelty.
- Avoid introducing hidden coupling or duplicate protocol surfaces without migration/deprecation notes.
- Validate that manifestation remains faithful to backend-decided state.

---
> Source: [FGD-ATR-EP/AGS](https://github.com/FGD-ATR-EP/AGS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
