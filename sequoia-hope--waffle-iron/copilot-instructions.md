## waffle-iron

> When rules conflict, the following precedence applies (highest first):

# Waffle Iron — Agent Team Structure

## Document Precedence

When rules conflict, the following precedence applies (highest first):

1. `/governance/*` — Engineering law (Constitution, FIP, DoD, Architectural Invariants)
2. `/agents/*` — Roles, skills, orchestration
3. This file — Repo-level team structure and directory scoping
4. `CLAUDE.md` — Session workflow and coding conventions
5. Sub-project `CLAUDE.md` files — Project-specific instructions

## Overview

Waffle Iron is developed by autonomous Claude Code agent teams. This document defines the **project-level** team structure: which agents own which directories, how they coordinate across crates, and what boundaries they must respect.

For **feature-cycle** roles (how work is executed within a feature), see `/agents/roles/` and `/governance/FEATURE_IMPLEMENTATION_PROTOCOL.md`.

## How the Two Role Systems Compose

There are two complementary role systems:

### Project-Level Roles (this file)
Define **who owns what** across the repo: which agent touches which crate, who can modify INTERFACES.md, who runs integration tests.

### Feature-Cycle Roles (`/agents/roles/`)
Define **how a feature is built**: Manager orchestrates, Spec Writer produces specs, Test Author writes failing tests, Implementer codes, Adversary hardens. These roles enforce separation of concerns within a single feature's lifecycle per the Feature Implementation Protocol.

In practice, a Sub-Project Agent may assume different feature-cycle roles across sequential cycles (e.g., act as Implementer in one cycle, then Test Author in the next), but must **never** act as both Test Author and Implementer within the same feature cycle.

## Project-Level Roles

### Manager

- **Scope:** Orchestration and dispatch. Never writes modeling code directly.
- **Reads:** `/governance/*`, `/agents/*`, top-level ARCHITECTURE.md, INTERFACES.md, all sub-project PLAN.md files.
- **Responsibilities:**
  - Interpret user requests and classify change type.
  - Route to correct feature-cycle workflow (see `/agents/ORCHESTRATION.md`).
  - Enforce Feature Implementation Protocol and Definition of Done.
  - Review cross-project interface compliance.
  - Prioritize work based on dependency graph.
  - Resolve inter-project conflicts.

### Sub-Project Agents (one per sub-project)

- **Scope:** Single sub-project directory only.
- **Reads:** Their project's ARCHITECTURE.md, PLAN.md, INTERFACES.md, CLAUDE.md, plus top-level INTERFACES.md.
- **Responsibilities:**
  - Implement code within their crate/directory.
  - Run their tests before every commit.
  - Update their PLAN.md to mark completed tasks and add discovered tasks.
  - Document interface change requests (never modify top-level INTERFACES.md directly).
  - Assume feature-cycle roles as directed by Manager (respecting role separation).

### Integration Agent

- **Scope:** Full workspace. Runs after sub-project milestones.
- **Reads:** All documentation. All code.
- **Responsibilities:**
  - Full workspace `cargo build` and `cargo test`.
  - Cross-crate integration tests.
  - Verify interface compliance across crate boundaries.
  - File issues when interfaces are violated.
  - Update top-level INTERFACES.md when approved changes are needed.

### Review Agent

- **Scope:** Read-only review across all sub-projects.
- **Reads:** All documentation and code.
- **Responsibilities:**
  - Interface compliance review (do crates use the types from INTERFACES.md?).
  - Test coverage review (are public functions tested?).
  - Documentation accuracy review (does PLAN.md reflect actual state?).
  - Determinism review (any non-deterministic code?).
  - Security review (any unsafe code, panics in production paths?).

## Rules

### Boundary Rules

1. **Sub-project agents NEVER modify files outside their sub-project directory.** Exception: Integration Agent.
2. **Sub-project agents NEVER modify top-level INTERFACES.md.** They document requested changes in their PLAN.md under "Interface Change Requests."
3. **No agent modifies another agent's branch** without explicit coordination.

### Workflow Rules

4. **Every agent reads INTERFACES.md before starting work.** Interface types are the contracts.
5. **Every agent runs tests before committing.** `cargo test -p <crate>` for sub-project agents. Full `cargo test` for Integration Agent.
6. **Every agent updates PLAN.md** to mark completed tasks and add discovered tasks.
7. **If stuck for more than 15 minutes without a commit,** the task scope is too broad. Break it down, document in PLAN.md, move on.

### Interface Change Process

8. Sub-project agent discovers interface gap → documents in their PLAN.md under "Interface Change Requests" with rationale and proposed change.
9. Manager reviews interface change requests across all sub-projects.
10. Integration Agent implements approved changes to top-level INTERFACES.md.
11. All consuming sub-project agents are notified and must update their code.

### Quality Rules

12. **Tests are permanent.** Passing tests must never be deleted.
13. **Determinism is mandatory.** Same inputs → same outputs. No random values, no system time, no non-deterministic iteration.
14. **Mock before real.** Test against MockKernel first, WaffleKernel second.
15. **Document failures.** If a kernel operation fails in unexpected ways, document it in the kernel PLAN.md.

---
> Source: [sequoia-hope/waffle-iron](https://github.com/sequoia-hope/waffle-iron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
