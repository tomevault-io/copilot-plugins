## phantom-reasoning

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Phantom Reasoning** trains LLMs with GRPO (Group Relative Policy Optimization) on synthetic multi-hop reasoning datasets (PhantomWiki, GSM-Infinite, ReasoningGym) and evaluates transfer to real-world multi-hop datasets (HotpotQA, 2Wiki, Musique, CofCA, SynthWorlds-RM).

## Common Commands

### Installation

```bash
uv pip install -e ".[dev]"
uv pip install flash-attn --no-build-isolation
pre-commit install
```

### Code Quality

```bash
# Run all pre-commit hooks on all files
pre-commit run --all-files

# Run on specific file
pre-commit run --files src/phantom_reasoner/grpo.py
```

### Training

```bash
# Generate cluster-specific training script (anvil, aida, empire)
./scripts/create_train_grpo__vllm_colocate.sh <cluster_name>

# Run GRPO training (after generating the .sub script above)
./scripts/train_grpo__vllm_colocate.sub \
    recipes/accelerate_configs/zero1.yaml \
    recipes/Qwen/Qwen3-1.7B/grpo/config_pw_4gpu.yaml
```

### Evaluation

```bash
# Real-world wiki datasets (hp500, 2wiki500, msq500, cofca500, synthrm500)
MODEL_NAMES="Qwen/Qwen3-1.7B" bash scripts/eval/wiki_eval_grpo.sh out__eval=wiki hp500 minidev

# PhantomWiki (requires phantom-wiki[eval] installed from ../phantom-wiki/)
MODEL_NAMES="Qwen/Qwen3-1.7B" bash scripts/eval/pw_eval_grpo.sh out__eval=pw

# GSM-Infinite
MODEL_NAMES="Qwen/Qwen3-1.7B" bash scripts/eval/gsminf_eval_grpo.sh out__eval=gsminf

# Evaluate all intermediate checkpoints
bash scripts/eval/wiki_eval_all_ckpts.sh <checkpoint_dir> hp500 minidev Qwen/Qwen3-1.7B pw
```

## Architecture

### Core Training (`src/phantom_reasoner/`)

- **`grpo.py`** — Main training script. Defines reward functions (`reward_f1`, `reward_exact_match`, `reward_binary_format`, etc.), loads datasets, and launches training via `CustomGRPOTrainer`. Reward functions take `completions`, `answer`, and `prompt_method` as arguments.
- **`datasets_for_grpo.py`** — Dataset classes: `PhantomWikiDataset`, `GSMInfiniteDataset`, `ReasoningGymDataset`, `HotpotQADataset`, `TwoWikiDataset`, `MuSiQueDataset`. Each returns formatted prompt/answer pairs for GRPO.
- **`configs.py`** — `GRPOConfig` and `GRPOScriptArguments` dataclasses that extend TRL's config with custom flags like `prompt_method`, `reward_func_names`, `data_curriculum`.
- **`trainers/custom_grpo_trainer.py`** — Extends TRL's `GRPOTrainer` with custom logging and checkpoint management.
- **`_types.py`** — Shared type aliases (e.g., `CONVO_T` for conversation format).

### Evaluation Pipeline (`examples/`)

- **`examples/wiki/pred.py`** — Runs inference using a vLLM server; writes predictions to CSV.
- **`examples/wiki/format_split_accuracy.py`** — Aggregates per-sample CSVs into summary metrics.
- **`examples/wiki/create_bar_plots_performance_transfer.py`** — Bar charts comparing base vs. GRPO-trained models on real-world datasets.
- **`examples/wiki/plot_all_wiki_scaling_final_ckpts.py`** — F1 vs. training steps plots.

### Recipes (`recipes/`)

YAML configurations per model and dataset (e.g., `recipes/Qwen/Qwen3-1.7B/grpo/config_pw_4gpu.yaml`). Key fields: `model_name_or_path`, `reward_func_names`, `prompt_method` (`cot` or `zeroshot`), `data_curriculum`, vLLM settings. Accelerate configs (`zero1.yaml`, `zero2.yaml`, `zero3.yaml`) control DeepSpeed ZeRO stages.

### Data Flow

Training reads from `./data/` (symlinked from cluster shared storage). Evaluation writes predictions to output directories specified by `out__eval=<name>` or `out__train=<name>__eval=<name>`. Final checkpoint paths are listed in `scripts/final_plots/final_ckpts.yaml`.

## Environment Variables

`SCRATCH` must be set in the shell before running `scripts/setup_conda_env_vars.sh`. All other variables are set via `conda env config vars set` and persist across sessions:

| Variable           | Purpose                                                                                      |
| ------------------ | -------------------------------------------------------------------------------------------- |
| `SCRATCH`          | Base directory for experiments (must be pre-set in shell)                                    |
| `CONDA_ENV_NAME`   | Active conda environment name                                                                |
| `RUN_BASE_DIR`     | Base directory for checkpoints (`$SCRATCH/phantom-reasoning`, or `.` if `$SCRATCH` is unset) |
| `HF_HOME`          | HuggingFace cache directory (`$SCRATCH/huggingface`; only set when `$SCRATCH` is set)        |
| `ANVIL_PROJECT_ID` | Anvil cluster project ID (only set when `cluster=anvil`)                                     |
| `USER_EMAIL`       | Email for SLURM job notifications                                                            |
| `WANDB_ENTITY`     | W&B organization                                                                             |
| `WANDB_PROJECT`    | W&B project name (defaults to `phantom-reasoning`)                                           |

## Code Style

### Python

- **Python 3.12** required. Use `uv run python` to execute scripts.
- Add shebangs to standalone scripts: `#!/usr/bin/env -S uv run python`
- Add module-level docstrings to all scripts describing purpose and usage.
- Use function docstrings with `Args:` and `Returns:` sections when the signature is not self-evident.
- Use `typing` module only when necessary (prefer built-in generics: `list[str]`, `dict[str, int]`).
- Formatting: **Black** (line length 110), **isort** (black profile), **flake8** (max-complexity 10).
- Imports: sorted by isort; unused imports removed by autoflake.

### Bash

- Shebang: `#!/usr/bin/env bash` and make the script executable.
- Add a comment block at the top describing what the script does.
- Include a usage printout when when missing required args.
- Use `cmd_args=$@` after shifting required bash script arguments to collect and pass through command-line arguments.

---
> Source: [kilian-group/phantom-reasoning](https://github.com/kilian-group/phantom-reasoning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
