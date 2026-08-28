## simthesizer

> This repository's primary documents are listed below. Skim this list first when starting work, then read the relevant document(s) based on your goal.

# Project's Guideline

## Documents Index

This repository's primary documents are listed below. Skim this list first when starting work, then read the relevant document(s) based on your goal.

- `ARCHITECTURE.md`: Architecture boundaries and invariants of the current implementation. Read before changing core structure.
- `.agents/skills/`: Agent skills for this repository. `sim-dev` holds simulator modeling guardrails (read before changing `crates/simthesizer-core/`); `task-design` guides writing task specs for implementation agents. Claude Code picks these up through the `.claude/skills` symlink; Codex reads `.agents/skills/` directly.

## ARCHITECTURE.md

This document is the living specification for this repository’s architecture. Its job is to give contributors a *correct mental model* before they change core structure.

### Scope
- Describe stable boundaries, invariants, and cross-cutting concerns.
- Prefer what must stay true over implementation walkthroughs.
- Avoid duplicating code comments or listing every module.

### Maintenance (non-negotiable)
- If a change affects boundaries, invariants, major dataflows, or cross-cutting concerns, update this file in the same change set.
- After executing an ExecPlan, this file **must** be updated to reflect the post-plan architecture (it must stay up to date).
- If this document becomes wrong, fix correctness before expanding detail.

### Writing rules
- Start with a “Bird’s Eye View”: inputs, outputs, major subsystems, and the main loop/dataflow.
- Maintain a “Code Map”: subsystem → canonical path mapping (navigation index).
- Use explicit callouts:
  - **Architecture Invariant:** a falsifiable statement that must remain true (including things deliberately absent).
  - **API Boundary:** an edge where rules differ (dependencies, IO, error model, serialization, ownership, performance).
- For each invariant, prefer an enforcement story: tests, dependency rules, or an explicit review checklist item.

### Suggested sections (as code appears)
1. Bird’s Eye View
2. Code Map
3. Architecture Invariants
4. API Boundaries
5. Cross-Cutting Concerns (testing, error handling, performance, observability, code generation, cancellation)
6. Decision Log (architecture-shaping decisions only; date + rationale)

### Review checklist
- Does this file still match the code *after* the change?
- Did we add/modify any **Architecture Invariant** or **API Boundary**?
- Can a new contributor find the correct entry points via the Code Map?

---
> Source: [casys-kaist/Simthesizer](https://github.com/casys-kaist/Simthesizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
