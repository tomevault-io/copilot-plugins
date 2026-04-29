## agent-harness-eval

> - Reason from invariants first. Prefer the smallest correct design over patching special cases.

# Repository Guidelines

## Working Approach

- Reason from invariants first. Prefer the smallest correct design over patching special cases.
- Clarify requirements before building when ambiguity would cause real rework.
- Fail fast on invalid config, unsupported states, and broken contracts.
- Keep behavior changes and doc updates in the same change.
- Do not preserve compatibility when a cleaner internal interface is better; this project is pre-1.0.
- Solve the root problem instead of stacking one more special-case branch.

## Design Principles

- Prefer explicit, observable, and testable designs over clever abstractions.
- Keep interfaces small and ownership clear.
- Add only the minimum code and configurability needed for the current requirement.
- Follow existing naming patterns within the same layer instead of inventing a new variant.

## Python Conventions

- Target Python 3.12+.
- Use `uv` for environment and dependency management.
- Keep runtime configuration in `src/agent_harness_eval/config/`.
- Keep harness-specific protocol handling in `src/agent_harness_eval/adapters/`.
- Keep grading logic independent from harness implementations.

## Testing

- Keep tests isolated and filesystem-local.
- Add or update tests for behavior changes.
- After meaningful eval or end-to-end runs, separate test-design issues from real regressions before proceeding.
- Run before finishing:
  - `.venv/bin/ruff check src/agent_harness_eval tests`
  - `.venv/bin/pytest tests`

## Changes And Reviews

- Keep changes minimal and maintainable.
- When reviewing or debugging, prioritize bugs, regressions, missing tests, and weak assumptions over style commentary.
- Note config or migration impact when behavior changes.
- When creating a git commit, include a descriptive body in addition to the one-line subject. Do not use a subject-only commit message; the body should briefly explain what changed and any important context or verification.


## Local Files

- Do not commit local secrets or machine-specific files such as `.env`, `eval.yaml`, `.harnesses/`, `results/`, or cache directories.
- `eval.yaml.example` is the public template; `eval.yaml` is local developer state.

---
> Source: [hellock/agent-harness-eval](https://github.com/hellock/agent-harness-eval) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
