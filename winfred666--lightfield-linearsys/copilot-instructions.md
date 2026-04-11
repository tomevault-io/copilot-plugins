## lightfield-linearsys

> This file is the contributor workflow contract for this repo. It defines how we structure code/data, how we track work, and how we verify changes.

# Project instructions: FISTA solver for massive sparse linear systems

This file is the contributor workflow contract for this repo. It defines how we structure code/data, how we track work, and how we verify changes.

## Scope & goal

- Goal: implement and iterate on solvers (baseline: FISTA) for the large linear system described in `README.md`.
- Keep changes small, testable, and reversible.

## Repository conventions

- `src/`: all Python source code (installable package via `pyproject.toml`).
- `scripts/`: one-off utilities (data checks, conversions). These should import from `src/` when possible.
- `data/`: raw + processed data (do not commit large binaries unless explicitly intended).
	- Recommended layout (already used in the repo):
		- `data/raw/`: original input files
		- `data/processed/`: preprocessed tensors/datasets (e.g., `pair_*.pt`)
- `result/`: outputs produced by experiments/runs.
- `test/`: pytest tests.

## Task agreement (how we work)

1. Single source of truth: Current status, TODOs, and data description live in `README.md`.
2. Iterate in small steps: Don't try to finish every TODO in one pass. If a change is large, split it into multiple PR-sized steps.
3. Keep docs in sync: If you change file structure, APIs, preprocessing, or data assumptions, update `README.md` immediately.
4. Write a short Develop Log entry: After finishing a meaningful step (new module, new experiment, solver improvement), add a brief note under `## Develop Log` in `README.md`:
	 - what changed
	 - observed results (timing/quality/metrics)
	 - next hypothesis / next step

## Environment & dependencies

- Use the existing conda environment named `torch` for running code.
- Dependencies must be declared in `pyproject.toml`.
	- Runtime deps go under `[project].dependencies`.
	- Dev/test deps go under `[project.optional-dependencies].dev`.

## Verification standard

For each new module or behavior change:

1. Add/Update tests in `test/` using `pytest`.
	 - Include at least:
		 - one minimal "happy path" test (e.g., small synthetic Ax=b)
		 - one edge case (shape mismatch, zero rows, sparse tensor input, etc.)
2. Quick local check should run in a few seconds on Linux.
3. Prefer tiny synthetic problems for unit tests; keep real-data runs in scripts.

## Coding guidelines

- Prefer pure functions with clear input/output shapes.
- Document tensor axis conventions explicitly (e.g., volume is `(X, Y, Z)`; image is `(Y, X)`).
- When touching preprocessing, add assertions and informative errors around shape alignment.
- Avoid hard-coded absolute paths; use project-relative paths (e.g., `data/raw/...`).
- All disposable shells are in data/xxx.sh or data/xxx.py so that it won't be included in our repo.


## Notes

- If instructions here conflict with `README.md`, treat `README.md` as authoritative and update this file to match.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Winfred666)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Winfred666)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
