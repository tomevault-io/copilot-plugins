## auto-aicoding-harness

> This repository is the source of truth for `Auto_AICoding_harness`.

# AGENTS.md

## Purpose

This repository is the source of truth for `Auto_AICoding_harness`.
It is not a generated target project.

## Default Working Rules

Before making non-trivial changes:

1. read `README.md`
2. read `docs/design/reviewed-final-design-v1.md`
3. read any contract file directly affected by the change

## Repository Boundaries

- `templates/` contains template source only
- `profiles/` contains engineering overlays only
- `docs/design/` contains workflow and repository contracts
- root live `.ai/`, `.codex/`, `.agents/` are not product structure and should stay ignored if created locally

## Editing Rules

- Do not move target-project runtime files into the harness root.
- Do not couple core workflow contracts to `C++`, `Linux`, `CMake`, or other profile policy.
- Do not create duplicate truths for generated files outside `templates/`.
- Prefer small, local edits that preserve contract clarity.
- Apply `karpathy-guidelines` as the default behavioral lens for planning, code changes, reviews, and refactors in this repository.
- Unless the user explicitly says not to ask, do not silently choose between materially different implementations.
- Ask targeted clarification questions when direction, scope, acceptance criteria, or risk boundaries are ambiguous, but avoid performative questioning that would not change the work.

## Verification

For doc and skeleton changes, verify by:

1. checking repository structure
2. checking cross-file consistency of command, state, and boundary terms
3. checking `.gitignore` behavior for root runtime directories

---
> Source: [yu20120707/Auto_AICoding_Harness](https://github.com/yu20120707/Auto_AICoding_Harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
