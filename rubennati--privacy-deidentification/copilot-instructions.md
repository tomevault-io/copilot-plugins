## privacy-deidentification

> Before starting work, always read:

# Agent Instructions

Before starting work, always read:

- `.ai/index.md`
- `.ai/state.md`
- `.ai/routing.md`

## Core rules

- Do not perform unrelated refactors.
- Request human approval before major architectural or dependency changes.
- Update `.ai/state.md` after meaningful changes.
- Record architectural decisions as ADRs under `docs/adr/` and link them in `.ai/decisions.md`.
- Never commit secrets. Configuration comes from environment variables.

## Engine maturity levels (0–19)

The central engines are planned on a **0–19 maturity scale** (see
[`docs/engine/`](docs/engine/README.md#maturity-scale) and
[ADR-0016](docs/adr/0016-engine-maturity-levels-0-19.md)):

- Use the 0–19 scale for OCR/Text, PII/Sensitive-Data, Review/Human-Feedback, Benchmark/Regression,
  and Redaction planning. Level numbers are cumulative within an engine and not comparable across
  engines.
- New engine PRs should state **which level they advance**, and PR summaries should mention the
  affected engine level where relevant.
- Do **not** mix the older 0–10/0–14 numbering without a migration note; each engine document has a
  *Legacy scale mapping* table to translate old citations.
- For *what* is detected and *how sensitive* it is — business categories, entity types, risk classes
  **P0–P5**, and detection strategies — use [`docs/engine/entity-taxonomy.md`](docs/engine/entity-taxonomy.md)
  ([ADR-0017](docs/adr/0017-entity-taxonomy-and-risk-classes.md)). New recognizer/entity-type work
  should name its category, risk class, and detection strategy.
- **Check the OCR/PII implementation plan before starting engine work**
  ([`docs/engine/ocr-pii-implementation-plan.md`](docs/engine/ocr-pii-implementation-plan.md),
  [ADR-0018](docs/adr/0018-ocr-pii-implementation-plan.md)). OCR/Text and PII/Sensitive-Data are the
  core engines; **do not let PII/Redaction outrun OCR/Text prerequisites** (OCR/Text stays 2–3 levels
  ahead). Every engine PR must state which 0–19 level it advances. After every engine PR, run the
  plan's checkpoint loop and update `.ai/state.md`/docs if the level or the next plan changed.
- Feature and documentation PRs target `dev`, not `main` (see [Approval](#approval)).

## Product principle: tool-first / adapter-bound

Core intelligence comes from **proven open-source tools behind ports/adapters**. Do not build a
bespoke OCR engine, NER/LLM engine, redaction engine, or pseudonymization engine. Also avoid large
opaque rule sets and untested ad-hoc heuristics.

Small, deterministic domain logic is allowed when it remains adapter-bound, documented, tested,
benchmarkable, reviewable, and auditable. Examples include Presidio `PatternRecognizer`s, context
rules, candidate validation, domain recognizers, and narrowly scoped deterministic heuristics.
External engines must remain replaceable without changing orchestration or product workflows.
Our code owns orchestration, adapters, review UI, file handling, export, and secure integration.

## Workflow

For any non-trivial task: **Understand → Plan → Implement → Verify → Review.** Trivial
changes (typos, one-line fixes, additive doc tweaks) may skip the Plan step but still go
through a branch + PR.

1. **Understand** — read the relevant `.ai/` files and the code/docs the task touches.
2. **Plan** — state the plan before editing; confirm scope for multi-file work.
3. **Implement** — focused changes on a short-lived branch, one concern per branch.
4. **Verify** — run the standard quality commands locally (below).
5. **Review** — open a PR with a clear summary; a human merges.

## Standard quality commands

Run via the `Makefile` (everything runs in Docker — no host toolchain needed):

- `make lint` — Ruff (Python) + ESLint (TypeScript)
- `make typecheck` — mypy (Python) + `tsc --noEmit` (TypeScript)
- `make test` — pytest (backend) + Vitest (frontend)
- `make build` — build both container images
- `make up` / `make down` — start / stop the stack

## Approval

"Human approval" means a human merging the pull request. Concretely:

- AI agents must target `dev` for feature PRs. `main` is the user-stable branch and only receives
  curated merges from `dev`; direct feature PRs to `main` are reserved for explicit hotfixes.
- Windows installer and update scripts must always use `main`.
- Agents may commit and push to non-`main` branches (e.g. `feat/*`, `fix/*`).
- Agents must not merge to `main`, force-push, or modify branch protection.
- Agents must not push directly to `main`, even where branch protection is not configured.

If a change is larger than the current task's scope (architecture, dependencies), pause and
ask before committing.

## Tool-specific pointer files

This file is the source of truth for all AI tools. `CLAUDE.md` is a thin pointer for Claude
Code and must defer to this file. Do not duplicate rules across files; add them here.

---
> Source: [rubennati/privacy-deidentification](https://github.com/rubennati/privacy-deidentification) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
