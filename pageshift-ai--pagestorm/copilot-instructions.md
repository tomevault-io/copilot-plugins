## pagestorm

> PageStorm owns the trained model protocol, bundle loading, output parsing, and

# PageStorm Agent Instructions

## Scope

PageStorm owns the trained model protocol, bundle loading, output parsing, and
the vLLM model adapter implementation.

- Keep ranking, refinement, application UI, and deployment orchestration in
  their owning repositories.
- Do not add local artifacts, credentials, generated books, model weights, or
  machine-specific paths.
- Do not depend on sibling repository checkouts.

## Working Rules

- Inspect the relevant code path before editing it.
- State assumptions when a task is ambiguous.
- Prefer the smallest change that satisfies the request.
- Do not add speculative abstractions or fallback paths.
- Do not refactor adjacent code unless the requested change requires it.
- Preserve the staged PageStorm role headers and structured-output contracts.
- Use `rg` for searches.

PageStorm checkpoints are staged completion models, not chat assistants. Do not
convert their prompts into chat messages or add correction turns.

## Environment

Use the repository-local environment:

```bash
uv venv --python 3.11
uv sync --extra dev
.venv/bin/pre-commit install
```

## Validation

Before reporting a code change as complete, run:

```bash
.venv/bin/pre-commit run --all-files
.venv/bin/python -m pytest
```

Fix failures caused by the change. Report any check that could not be run.

---
> Source: [Pageshift-ai/pagestorm](https://github.com/Pageshift-ai/pagestorm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
