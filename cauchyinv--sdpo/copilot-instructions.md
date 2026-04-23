## sdpo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SDPO (Self-Distilled Policy Optimization) is a reinforcement learning framework for LLMs that learns from verifiable rewards with rich environment feedback. It extends the `verl` framework with self-distillation: the model conditions on its own high-reward trajectories and environment feedback (error messages, test results) to generate dense learning signals without external teachers.

## Common Commands

### Running Experiments

```bash
# Run SDPO training directly (single machine, 8 GPUs)
bash experiments/rich_feedback/sdpo.sh [--dry-run] [--skip-install]

# Run GRPO baseline
bash experiments/rich_feedback/grpo.sh [--dry-run] [--skip-install]

# Submit to SLURM cluster
bash experiments/rich_feedback/run_sdpo.sh

# Local single-GPU testing
bash run_local_sdpo.sh [suffix]
bash run_local_grpo.sh [suffix]
```

### Data Preparation

```bash
# Download dataset (options: Chemistry, Biology, Material, Physics, livecodebench/code_generation_lite-v6)
python data/load_dataset.py --dataset_name Chemistry --output_path datasets/chemistry.json

# For LiveCodeBench (code generation)
python data/load_dataset.py --dataset_name livecodebench/code_generation_lite-v6 --output_path datasets/lcb_v6.json
python data/split_tests.py --json_path datasets/lcb_v6.json --output_dir datasets/lcb_v6

# For SciKnowEval
python data/split_tasks.py --json_path datasets/chemistry.json --output_dir datasets/chemistry --test_ratio 0.1

# Convert to parquet format required by trainer
python data/preprocess.py --data_source datasets/chemistry
```

### Running Tests

```bash
python -m pytest tests/ -v
pytest tests/workers/config/test_actor_config_on_cpu.py  # Specific test
```

### Training Entry Point

```bash
# Direct Hydra invocation
python -m verl.trainer.main_ppo --config-name sdpo \
  data.train_batch_size=32 \
  actor_rollout_ref.actor.self_distillation.alpha=1.0 \
  vars.task=datasets/lcb_v6
```

## Architecture

### Core Training Loop

`verl/trainer/main_ppo.py` → `verl/trainer/ppo/ray_trainer.py`

The trainer orchestrates:
1. **Rollout phase**: vLLM generates `rollout.n` solutions per prompt
2. **Reward scoring**: domain-specific functions in `verl/utils/reward_score/feedback/`
3. **Reprompting** (SDPO only): successful solutions + environment feedback are used to construct a new conditioning prompt
4. **Self-distillation loss**: KL divergence between student (plain prompt) and teacher (reprompt-conditioned) distributions
5. **Policy update**: Combined PPO + distillation loss

### Key Files

| File | Role |
|------|------|
| `verl/trainer/ppo/core_algos.py` | Loss functions: `compute_policy_loss_vanilla()`, `compute_self_distillation_loss()` |
| `verl/workers/config/actor.py` | `SelfDistillationConfig` dataclass with all SDPO hyperparameters |
| `verl/trainer/ppo/ray_trainer.py` | Distributed training loop |
| `verl/trainer/ppo/rollout_corr_helper.py` | Rollout correction and reprompting logic |
| `verl/utils/reward_score/feedback/` | Reward functions: `code.py`, `math.py`, `mcq.py`, `tooluse.py` |
| `verl/utils/tracking.py` | WandB integration |

### Configuration System

Uses **Hydra** for config management. Configs compose via `defaults`:
- `verl/trainer/config/sdpo.yaml` — SDPO hyperparameters, inherits from `ppo_trainer` + `user`
- `verl/trainer/config/baseline_grpo.yaml` — GRPO baseline config
- `verl/trainer/config/user.yaml` — **Edit this file** for local paths (`vars.dir`, `vars.log_dir`, `vars.ckpt_dir`, model path)
- `verl/trainer/config/ppo_trainer.yaml` — Base PPO config

### SDPO Key Parameters

Controlled via `SelfDistillationConfig` (in `verl/workers/config/actor.py`) and `sdpo.yaml`:

- `alpha`: KL interpolation — `0.0`=forward KL (peak-seeking), `1.0`=reverse KL (mass-covering), `0.5`=JSD
- `is_clip`: Importance sampling clip value (typically `2.0`; `None` disables)
- `teacher_regularization`: `"ema"` or `"trust-region"`
- `teacher_update_rate`: EMA rate (e.g., `0.05`)
- `success_reward_threshold`: Min reward to qualify as a successful demonstration (default `1.0`)
- `dont_reprompt_on_self_success`: Skip reprompting when the current rollout already succeeded
- `max_reprompt_len`: Token budget for reprompt context (default `10240`)
- `distillation_topk`: Use top-k logits only for efficiency (`None` = full distribution)
- `include_environment_feedback`: Include error messages/test output in reprompt

### Supported Domains

- **Code**: LiveCodeBench — code executed against unit tests; feedback = test output / error traces
- **Math**: Math datasets — symbolic answer verification
- **Science MCQ**: SciKnowEval (Biology, Chemistry, Material, Physics) — multiple-choice, L3 questions only
- **Tool Use**: Pre-split datasets in `datasets/tooluse/`

## Local Path Configuration

Edit `verl/trainer/config/user.yaml` to set:
- `vars.dir`: Repository root
- `vars.log_dir`: WandB / logging output
- `vars.ckpt_dir`: Checkpoint output directory
- `actor_rollout_ref.model.path`: Base model (default `Qwen/Qwen3-8B`)

## Installation

```bash
pip install torch==2.5.1 --index-url https://download.pytorch.org/whl/cu124
pip install -r requirements.txt
pip install -e .
# Optional: flash-attn (pre-built wheels in repo root)
pip install flash_attn-2.8.2+cu12torch2.5cxx11abiTRUE-cp312-cp312-linux_x86_64.whl
```

## Experiment Types

- `experiments/rich_feedback/` — Primary SDPO experiments with environment feedback
- `experiments/generalization/` — Tests on held-out tasks without rich feedback
- `experiments/ttt/` — Test-time training (Section 5 of paper)
- `baseline_multiturn/` — Multi-turn baseline comparison

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CauchyInv) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
