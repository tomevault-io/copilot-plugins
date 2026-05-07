## t3rl

> T3RL is a multi-turn tool-use RL training package built on [slime](3rdparty/slime) (v0.2.4). It trains LLMs to perform multi-turn function calling via GRPO using the BFCL evaluation environment. See [README](README.md) for background and [docs/architecture.md](docs/architecture.md) for internals.

# CLAUDE.md

## Project Overview

T3RL is a multi-turn tool-use RL training package built on [slime](3rdparty/slime) (v0.2.4). It trains LLMs to perform multi-turn function calling via GRPO using the BFCL evaluation environment. See [README](README.md) for background and [docs/architecture.md](docs/architecture.md) for internals.

## Setup

```bash
git submodule update --init --recursive
pip install -e .                           # t3rl + bfcl_eval (NOT slime)
pip install -e 3rdparty/slime --no-deps    # slime separately
pip install -e ".[dev]"                    # pytest>=8
```

## Commands

```bash
# Data preprocessing
python data/preprocess_bfcl_data.py --output_dir data/processed/bfcl

# Training (8x H200)
bash scripts/train/stage1.sh               # Stage 1
bash scripts/train/stage2.sh               # Stage 2
RESUME_EXPERIMENT=<name> bash scripts/train/stage1.sh  # Resume

# Evaluation
bash scripts/test/run_bfcl_multiturn_eval.sh           # Official BFCL eval
bash scripts/test/launch_sglang.sh                     # Launch SGLang server
bash scripts/test/run_t3rl_eval.sh                     # Standalone T3RL eval

# Debug
bash scripts/debug/debug_rollout.sh                    # Rollout only (no Megatron)
bash scripts/debug/debug_training.sh                   # Training from saved .pt

# Rollout viewer TUI (requires: pip install "textual==0.52.1" rich typer)
python scripts/rollout_viewer.py --rollout-dir /path/to/rollout_data

# Tests
pytest
```

## Architecture & Key Docs

- [Architecture](docs/architecture.md) — Runtime call chain, slime hooks, key modules, TITO invariant, parser delegation, reward signal, config reference, environment variables
- [Evaluation](docs/eval.md) — Standalone eval pipeline, eval_mode semantics, async design
- [Background](docs/background.md) — Paper techniques, curriculum, trained models
- [BFCL Multi-Turn Reference](docs/bfcl-multiturn-reference.md) — Dataset structure, scoring, execution engine, edge cases

## Critical Invariant: TITO

`len(rollout_log_probs) == len(loss_mask) == response_length` — must hold across all multi-turn episodes. `TokenTrajectory.append_response()` enforces via `ValueError`. See [TITO details](docs/architecture.md#tito-token-in-token-out-invariant).

## Design Notes

- **Lazy slime imports**: `slime` is imported inside functions / try-except so `t3rl` can be imported without slime (parser-only usage, unit tests).
- **Parsers are pure text transforms**: No env state or token trajectory mutation.
- **Config via YAML**: `configs/bfcl/` loaded at runtime; fields become attributes on the `args` object.

---
> Source: [IcyFish332/T3RL](https://github.com/IcyFish332/T3RL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
