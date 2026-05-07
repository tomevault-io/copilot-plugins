## codex-long-running-harness

> This repository implements a `Codex-first harness` for long-running application development.

# Codex Harness Repository Guide

## Mission

This repository implements a `Codex-first harness` for long-running application development.

The system is intentionally split into three execution roles:

- `planner`: expands the user goal into high-level product and design artifacts
- `generator`: implements one sprint at a time and produces a candidate revision
- `evaluator`: independently validates the candidate revision with executable checks

## Working Rules

- Treat [`IMPLEMENTATION_MANUAL.md`](IMPLEMENTATION_MANUAL.md) as the source of truth for structure and file responsibilities.
- Treat [`EXECUTION_POLICY.md`](EXECUTION_POLICY.md) as the source of truth for execution policy.
- Treat [`TODOLOG.md`](TODOLOG.md) as the source of truth for progress tracking.
- Keep run state in `.harness/runs/<run_id>/`.
- Do not invent hidden state that only exists inside a model conversation.
- Do not allow `generator` to self-certify final success.

## Engineering Constraints

- Prefer Python standard library unless an external package clearly improves schema validation or typing.
- Use JSON Schema for all major artifacts.
- Keep prompts in `prompts/` and reusable output shapes in `schemas/`.
- Keep browser-based acceptance checks in `evaluators/playwright/`.
- Keep repository-safe defaults. Secrets must not be committed.

## Definition Of Done

An implementation task is not complete until:

1. The code or document is written to disk.
2. Any necessary tests have been run, or the reason they were not run is explicitly recorded.
3. [`TODOLOG.md`](TODOLOG.md) has been updated.
4. Structural changes are reflected in [`IMPLEMENTATION_MANUAL.md`](IMPLEMENTATION_MANUAL.md) when required.

## Core Paths

- `orchestrator/`: Python orchestration logic
- `prompts/`: role prompts and shared constraints
- `schemas/`: structured artifact contracts
- `templates/`: reusable markdown templates and AGENTS snippets
- `evaluators/`: executable validation harnesses
- `.harness/`: runtime artifacts

---
> Source: [LongWeihan/codex-long-running-harness](https://github.com/LongWeihan/codex-long-running-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
