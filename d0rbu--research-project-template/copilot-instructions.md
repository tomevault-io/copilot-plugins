## research-project-template

> You are working in a correctness-first Python research template. The repo should stay

# AGENTS.md - research-project-template

You are working in a correctness-first Python research template. The repo should stay
small, explicit, and easy to reuse as the base for new research projects.

This file is the AI-agent entry point. It should point to docs that contain durable
project knowledge. If you want to add detail here, usually update the linked doc instead.

## Read these first

- [`README.md`](README.md) - project pitch and quickstart
- [`docs/README.md`](docs/README.md) - documentation map
- [`docs/onboarding/getting-started.md`](docs/onboarding/getting-started.md) - local setup
- [`docs/onboarding/workflows.md`](docs/onboarding/workflows.md) - common development flows
- [`docs/development/correctness.md`](docs/development/correctness.md) - correctness philosophy and tools
- [`docs/reference/architecture.md`](docs/reference/architecture.md) - package architecture
- [`docs/reference/configuration.md`](docs/reference/configuration.md) - tool configuration
- [`docs/reference/file-reference.md`](docs/reference/file-reference.md) - file-by-file reference

## Repo layout

```
tests/              pytest suite, including property tests
docs/               source-of-truth documentation
.github/workflows/  CI checks
```

## Conventions

- Use `uv sync` to install and `uv run ...` to invoke project tools.
- Run `uv run pre-commit run --all-files` before handoff.
- The pre-commit hooks enforce `uv lock --check`, `ruff`, `ty`, and `pytest`.
- Prefer making bad state unrepresentable over documenting invalid states after the fact.
- Use `phantom-types` for domain invariants that narrow primitive values.
- Use `beartype` at runtime boundaries where invalid values can enter the system.
- Use `jaxtyping` for array shape and dtype contracts.
- Use Hypothesis for invariants, edge cases, and regression tests that should hold over many inputs.
- Keep imports at the top of each file.
- Keep docs and code in sync; when behavior changes, update `docs/reference/file-reference.md`.

## Correctness tools

The scaffold tests demonstrate:

- `Probability`: a phantom type for closed-range probabilities.
- `normalize_weights`: a `jaxtyping` + `beartype` checked NumPy function.
- property tests that use `st.from_type(...)` with phantom types.

Copy these patterns for project-specific concepts such as dataset splits, feature IDs,
sample counts, model dimensions, or validated artifact paths.

## Testing

```bash
uv run pre-commit run --all-files
```

`pytest` is configured to collect from `tests/` and require 95% coverage on the
current scaffold tests. Update coverage `source` when the project grows real source
modules.

---
> Source: [d0rbu/research-project-template](https://github.com/d0rbu/research-project-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
