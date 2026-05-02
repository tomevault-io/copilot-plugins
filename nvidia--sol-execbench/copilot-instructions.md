## sol-execbench

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SOL ExecBench is NVIDIA's GPU kernel evaluation and benchmarking framework. It evaluates custom GPU kernel solutions (PyTorch, Triton, CUTLASS, cuDNN, CuTe DSL, cuTile, CUDA C++) for correctness and performance by comparing them against reference implementations.

## Build & Run Commands

```bash
# Install dependencies (uses uv package manager)
uv sync --all-groups

# Run the CLI
uv run sol-execbench <problem_dir> --solution solution.json

# Run all tests
uv run pytest tests/

# Run a single test file
uv run pytest tests/sol_execbench/core/data/test_definition.py

# Run tests matching a keyword
uv run pytest tests/ -k "test_correctness"

# Run sample integration tests (requires CUDA GPU)
uv run pytest tests/sol_execbench/samples/

# Lint and format
uv run ruff check .
uv run ruff format .
```

## Architecture

The codebase has three layers:

1. **CLI** (`cli/main.py`) — Click-based CLI entry point (`sol-execbench`). Parses args, invokes the driver, displays Rich tables with results.

2. **Driver** (`driver/`) — Orchestrates evaluation. `ProblemPackager` stages problem files (definition, workloads, solution sources) into a temp directory, then runs compilation and evaluation as **subprocesses**. The eval driver template (`driver/templates/eval_driver.py`, ~800 lines) is the self-contained script that runs inside the subprocess — it imports torch/triton, loads inputs, executes kernels, computes correctness/performance, and emits JSONL traces to stdout.

3. **Core** (`core/`) — Two sub-packages:
   - `core/data/` — Pydantic v2 models for all data types: Definition (kernel specs, tensor shapes), Solution (source files, build specs), Workload (input generation), Trace (evaluation results with correctness/performance).
   - `core/bench/` — Benchmarking utilities: CUDA event timing, numerical correctness computation, GPU clock locking, reward hack detection (detects timing manipulation, stream injection, monkey-patching).

### Key execution flow

CLI → ProblemPackager stages files to temp dir → subprocess compiles C++/CUDA (if needed) → subprocess runs eval_driver.py → eval_driver outputs JSONL traces to stdout → CLI parses traces and displays results.

### Two calling conventions

Kernel solutions use either **destination-passing style** (DPS, modifies pre-allocated outputs in-place) or **return-value style** (returns outputs directly). The eval driver handles both.

## Test Markers & Fixtures

- `@pytest.mark.requires_torch_cuda` — auto-skipped when no CUDA GPU available
- `@pytest.mark.cpp` — compiles C++/CUDA extensions (slow)
- `@requires_sm100` — skips on GPUs below sm_100 (Blackwell). Use this for cuTile tests.
- `tmp_cache_dir` fixture — isolated build cache per test via `SOLEXECBENCH_CACHE_PATH`

## Style

Ruff for linting and formatting. Rule E741 is ignored. Exclude `data/` and `tests/data/` from lint.

## Commits

Format: `#<Issue> - <Title>` with DCO sign-off (`git commit -s`).

---
> Source: [NVIDIA/SOL-ExecBench](https://github.com/NVIDIA/SOL-ExecBench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
