## churro

> This repo is `churro-ocr`, a Python 3.12+ OCR and page-detection toolkit for historical documents.

# AGENTS

This repo is `churro-ocr`, a Python 3.12+ OCR and page-detection toolkit for historical documents.

## Repo Layout

- `src/churro_ocr/`: library code and CLI entrypoints
- `tests/`: pytest suite and test assets
- `docs/`: Sphinx documentation source
- `scripts/` and `tooling/`: packaging, benchmarking, and evaluation helpers

## Preferred Workflow

Use Pixi from the repo root:

```bash
pixi install
pixi run format
pixi run lint
pixi run typecheck
pixi run test
pixi run docs-build
pixi run package-check
```

After each change, review docs to see if it needs updating and run the relevant Pixi tasks to check your work.

## Guardrails

- Prefer the Pixi tasks above over ad hoc commands.
- Keep library changes in `src/churro_ocr/` and add or update tests in `tests/`.
- Do not enable live integration tests unless the task explicitly calls for them and the required credentials are available.

---
> Source: [stanford-oval/Churro](https://github.com/stanford-oval/Churro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
