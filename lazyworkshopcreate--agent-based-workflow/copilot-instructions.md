## agent-based-workflow

> <!-- [PROJECT CUSTOMIZATION] Replace this section with your repository's purpose -->

# Copilot Instructions

## Repository Purpose

<!-- [PROJECT CUSTOMIZATION] Replace this section with your repository's purpose -->
<!-- Example: This repository is a standalone implementation of [product/system]. -->
<!-- Example: Reference material under `ref/` and archived docs are evidence inputs, not runtime dependencies. -->

## Active Working Set

1. `docs/design/` is the live technical design source of truth.
2. `docs/plans/` contains current implementation baselines and approved sequencing.
3. `docs/decisions/` contains accepted constraints.
4. `docs/status/` contains active closure items.
5. `docs/reference/` contains long-lived technical evidence.
6. `docs/archive/` is historical only unless the user explicitly asks for historical research.
7. All active documentation must be written in English.

## Code And Architecture Boundaries

<!-- [PROJECT CUSTOMIZATION] Replace the rules below with your project-specific architecture constraints -->

1. Keep service boundaries aligned with `docs/design/`.
2. Keep runtime behavior aligned with documented design patterns.
3. Treat source directories as separate concerns; do not collapse unrelated responsibilities together.
4. Keep tenant routing, authorization, auditability, and error semantics explicit for every new endpoint or workflow.

## Design Documents and Traceability

1. Treat `docs/design/` as the coding-ready contract source for DTOs, validation, authorization, tenant scope, error behavior, and service-specific closure items.
2. Every new endpoint or write workflow must be traceable to at least one data source and one behavior source; active design conclusions must be written into `docs/design/` before coding starts.
3. If traceability is incomplete, record a closure item in the relevant design document; route it to `docs/status/outstanding-items.md` only when repository-wide or cross-service.
4. Consolidate temporary review, validation, and discovery packs into active docs and archive them once their conclusions are absorbed.

## Document Rules

1. Use stable filenames without date prefixes in `docs/design/**`, `docs/status/**`, and `docs/reference/**`.
2. Use `YYYYMMDD_` prefixes in `docs/decisions/**`, `docs/plans/**`, and `docs/archive/**`.
3. Prefer templates in `docs/templates/` when creating or rewriting formal artifacts.
4. After renaming, moving, or archiving documents, update README entries and in-body references.
5. Do not treat `docs/archive/**` as the default working set.
6. Keep root `README.md` stable unless the user explicitly asks for repository-overview changes.
7. Keep root `AGENTS.md` and `CLAUDE.md` limited to static repository facts; behavioral instructions belong in `.github/copilot-instructions.md`.

## Implementation Expectations

1. Design before code for medium or large changes. Update the relevant document under `docs/design/` before or alongside code changes.
2. Service or module contracts must define request and response shape, validation, authorization, tenant scope, error behavior, and test expectations before implementation starts.
3. Prefer small, reviewable increments grounded in one domain or one capability family at a time.
4. When an implementation plan is expressed as delivery slices, require one completed slice per git commit unless the user explicitly asks for a different commit strategy.

## Verification And Debugging Expectations

1. Do not claim a change is complete, fixed, or passing without fresh verification evidence from the current run.
2. When verification fails or behavior is unexpected, stop speculative fixes and use root-cause-first debugging before changing more code.
3. Prefer worktree-first or other isolated branch setup for larger implementation or plan-execution runs so the main workspace stays stable.
4. Parallelize only when tasks are genuinely independent and do not share files, runtime state, or ordered slice dependencies.
5. For code-focused feature or bugfix work with a practical automated harness, prefer test-first changes and make the verification path explicit.

---

## Working Method Overview

| Asset | Location | Purpose | When to Use |
| --- | --- | --- | --- |
| **Prompts** | `.github/prompts/*.prompt.md` | One-shot focused entry points for narrow tasks | Task is clear, output is a single deliverable |
| **Skills** | `.github/skills/*/SKILL.md` | Multi-step repeatable workflows with staged reasoning | Task needs sequenced stages, bundled references, or subagent coordination |
| **Agents** | `.github/agents/*.agent.md` | Parent-coordinated worker subagents | A skill or prompt needs to delegate a bounded worker step |
| **Templates** | `docs/templates/*.md` | Standard document structures for formal artifacts | Creating or rewriting design, plan, decision, or evidence documents |

> **Agents are never user-facing starting points.** They are always delegated to by a parent skill or prompt. When the right entry point is unclear, use `.github/workflow-decision-tree.md`.

---

## Delivery Lifecycle

This repository follows a **12-stage** staged delivery lifecycle. Full stage detail, cross-cutting capabilities, subagent reference, and template reference: see [`.github/workflow-lifecycle.md`](.github/workflow-lifecycle.md).

**Stages:** Research (1) → Discover Requirements (2) → Clarify Scope (3) → Gather Evidence (4) → Technical Spike (5) → Design (6) → Plan Delivery (7) → Break Down Slices (8) → Plan Verification (9) → Execute (10) → Review (11) → Document Maintenance (12).

**Cross-cutting:** Systematic Debugging · Test-Driven Development · Parallel Agent Dispatch · Isolated Workspace (worktrees) · Mermaid Diagrams · AI Image Generation.

**Subagents** (parent-coordinated, never user-facing): Evidence Scout · Design Delta Drafter · Slice Boundary Auditor · Delivery Slice Executor · Verification Gate Runner · E2E Acceptance Runner · Document Ref Cleanup Executor.

**Templates** (`docs/templates/`): RESEARCH_FINDINGS · ANALYSIS_DOCUMENT · REQUIREMENTS_DOCUMENT · REQUIREMENT_CLARIFICATION · EVIDENCE_TRACEABILITY · TECHNICAL_SPIKE · TECHNICAL_DESIGN · SERVICE_DESIGN · API_CONTRACT · DECISION_RECORD · IMPLEMENTATION_PLAN · FEATURE_IMPLEMENTATION_BREAKDOWN · STAGE_DELIVERY_TASK_LIST · TEST_BREAKDOWN · DOC_MAINTENANCE_CHECKLIST · REFERENCE_DOCUMENT.

---

## Workflow Selection Quick Reference

When the right entry point is unclear, use `.github/workflow-decision-tree.md`. The abbreviated decision logic:

1. **Broad topic needing investigation?** → `discovery-workflow` (module: `research`) / `Conduct Research`
2. **Research findings need requirements definition?** → `discovery-workflow` (module: `requirements`) / `Discover Requirements`
3. **Ambiguous scope?** → `discovery-workflow` (module: `clarification`) / `Clarify Scope`
4. **Need evidence map?** → `discovery-workflow` (module: `evidence`) / `Trace Evidence`
5. **Technical uncertainty?** → `discovery-workflow` (module: `spike`) / `Research Spike`
6. **Service needs full lifecycle advancement?** → `design-workflow` (module: `service-delivery`)
7. **Need design draft?** → `design-workflow` (module: `technical-design` or `api-contract`) / `Draft Technical Design` / `Design Service` / `Design API Contract`
8. **Design stable, need delivery plan?** → `Plan Implementation`
9. **Plan exists, need slices?** → `planning-workflow` (module: `feature-breakdown`) / `Break Down Implementation`
10. **Need standalone verification planning?** → `planning-workflow` (module: `test-breakdown`) / `Break Down Verification`
11. **Approved plan, need coordinated execution?** → `execution-workflow` (module: `implementation`) / `Run Plan`
12. **One approved slice, ready to code?** → `execution-workflow` (module: `implementation`) / `Run Slice`
13. **Need full E2E or acceptance test run with triage?** → `execution-workflow` (module: `e2e-acceptance`) / `Run E2E Acceptance`
14. **Document maintenance only?** → `document-lifecycle-workflow` / `Maintain Docs`
15. **Failing test or unexpected behavior?** → `execution-workflow` (module: `debugging`)
16. **Independent parallel tasks?** → `execution-workflow` (module: `parallel-agents`)
17. **Need isolated workspace?** → `execution-workflow` (module: `worktrees`)
18. **Code-focused bugfix with test harness?** → `execution-workflow` (module: `tdd`)
19. **Need a Mermaid diagram rendered as PNG/SVG?** → `generate-mermaid-flowchart`
20. **Need an AI-generated cover image or illustration?** → `generate-ai-image`
21. **Review feedback to process?** → `Process Review Feedback`
22. **Ready for review handoff?** → `Request Code Review`

---

## Lifecycle Extension Points

This delivery lifecycle covers the design-to-code-to-review cycle. The following concerns are intentionally outside the default scope but can be added as project-specific stages, prompts, or skills when needed:

- **Deployment and release management** — CI/CD pipeline planning, release notes, rollback strategy.
- **Migration planning** — database schema changes, data migration, backward compatibility.
- **Monitoring and observability** — production alerting, tracing, health checks.
- **Incident response** — runbooks, escalation procedures, post-mortem templates.
- **Security review** — threat modeling, penetration testing, compliance checks.

Add domain-specific skills or prompts to cover these areas when your project requires them.

---

## Style

1. Be concise and concrete.
2. Prefer explicit boundaries, evidence, and closure items over vague summaries.
3. Challenge unsupported assumptions directly.
4. Unless explicitly requested, all produced artifacts must be written in English—this includes documentation, code comments, identifiers, log messages, and error text.

---
> Source: [LazyWorkshopCreate/agent-based-workflow](https://github.com/LazyWorkshopCreate/agent-based-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
