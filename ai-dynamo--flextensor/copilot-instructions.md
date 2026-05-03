## flextensor

> PyTorch tensor offloading library — GPU ↔ CPU for large models on limited GPU memory.

<!--
# SPDX-FileCopyrightText: Copyright (c) 2025-2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-License-Identifier: Apache-2.0
-->
# FlexTensor Development Guidelines

PyTorch tensor offloading library — GPU ↔ CPU for large models on limited GPU memory.

---

## Stack

Python 3.10+ | PyTorch >=2.5 (CUDA) | beartype | pydantic | psutil | scipy | ruff | mypy strict | pytest | uv

---

## Project Structure

```text
src/flextensor/
├── offload_manager.py         # High-level API (recommended)
├── tensor_manager.py          # Low-level API (advanced)
├── config.py                  # OffloadConfig
├── lazy_model_init.py         # Model init from saved profiles
├── state_handler.py           # State persistence/serialization
├── strategy/                  # Offloading strategies (knapsack, adaptive, global, etc.)
├── shm/                       # Shared memory subsystem (cross-process)
├── contrib/vllm/              # vLLM integration (worker, snapshots)
├── instrumentation/           # Diagnostics, dumpers, host resources
├── benchmark_tensor_mode.py   # Profiling state machine
├── trap_tensor_mode.py        # Tensor trap mechanism
├── tensor_discovery.py        # Module/tensor discovery
└── collectors.py              # Statistics collector utilities
tests/
├── unit/                      # Fast tests, no GPU required
└── integration/L{0,1,2}_*/   # GPU tests by complexity level
```

**Architecture**: `OffloadManager` (singleton) → `TensorManager` → Strategies (`Knapsack`/`Greedy`/`NthLayer`/`Adaptive`/`Global`)

**State Machine**: `NOT_INITIALIZED` → `DISCOVERY` → `PROFILING` → `INFERENCE`

---

## Commands

```bash
# Setup
uv venv --python 3.10 .venv && source .venv/bin/activate
uv pip install -e ".[dev]" && pre-commit install

# Unit tests (no GPU)
pytest -sv tests/unit --cov=flextensor --cov-report=term-missing

# Integration tests (GPU required, per level)
TEST_LEVEL=L0  # L0, L1, L2
for dir in tests/integration/*/; do
  [ -f "${dir}test.sh" ] && echo "$dir" | grep -qE "$TEST_LEVEL" && bash "${dir}test.sh"
done

# Quality
pre-commit run -a
uvx ruff check . && uvx ruff format .
uvx mypy src/flextensor/
```

---

## Worktrees

Each worktree needs its own `.venv` — the editable install is path-specific:

```bash
cd .worktrees/<branch>
uv venv --python 3.10 .venv && source .venv/bin/activate
uv pip install -e ".[test]"
```

---

## Conventions

- **Line length**: 120 chars
- **Docstrings**: Google-style (Args, Returns, Raises, Example)
- **License header**: SPDX in all owned source files — `.py`: `#` comment, `.md`/`.html`: `<!-- -->`, `.css`: `/* */`
- **Type hints**: Required everywhere; beartype for runtime
- **Coverage**: Minimum 80%

**Commits**: `feat(scope):` / `fix(scope):` / `docs:` / `test:` / `refactor:` / `chore:` — `feat!:` / `fix!:` for breaking changes, `BREAKING CHANGE:` footer for details

**Branches**: `###-feature-name` or `username/feature-name`

**Pull/merge requests**: One topic per PR/MR — don't mix unrelated fixes into the same branch.

**Changelog**: Update `CHANGELOG.md` under `## [Unreleased]` for user-visible changes. Follows [Keep a Changelog](https://keepachangelog.com/).

---

## Design Principles

- Composition over inheritance for strategies
- `Protocol` over `ABC` when runtime checking isn't needed
- Strategies focused on single algorithms
- Public API (`__all__`) clearly separated from internals
- No config options until users request them

---
> Source: [ai-dynamo/flextensor](https://github.com/ai-dynamo/flextensor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
