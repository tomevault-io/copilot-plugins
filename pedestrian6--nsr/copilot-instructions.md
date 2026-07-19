## nsr

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **RLVR** — a research implementation of *"The Surprising Effectiveness of Negative Reinforcement in LLM Reasoning"* (NeurIPS 2025). It implements PSR (Positive), NSR (Negative), and W-REINFORCE (Weighted) advantage algorithms on top of the **veRL** framework for training Qwen2.5-Math models on mathematical reasoning tasks.

The key insight: NSR trains exclusively on *incorrectly answered* prompts (negative advantage), PSR on *correctly answered* prompts, and W-REINFORCE blends them via `positive_advantage_weight`.

## Common Commands

### Installation
```bash
conda create -y -n verl python=3.10.14 && conda activate verl
pip install -e .
pip install vllm==0.8.2 latex2sympy2 fire "tensordict==0.7.2"
python -m pip install flash-attn --no-build-isolation
```

### Training
```bash
# PSR / NSR / W-REINFORCE (edit algorithm.advantage in script: positive | negative | weighted)
bash run_qwen2.5-math-7b_psr_nsr.sh

# PPO baseline
bash run_qwen2.5-math-7b_ppo.sh

# GRPO
bash run_qwen2.5-math-7b_grpo.sh

# Direct entry point (all scripts call this)
python3 -m verl.trainer.main_ppo <hydra overrides>
```

### Evaluation
```bash
python eval.py \
  --model_name="<checkpoint_path>" \
  --datasets="TianHongZXY/AIME2025,TianHongZXY/amc23,TianHongZXY/MATH" \
  --split="test" \
  --output_dir="eval_results/" \
  --batch_size=1000 --max_tokens=4096 --num_gpus=4 --num_generation=256

python calculate_metrics.py --file_path <output.jsonl>
```

### Code Formatting
```bash
bash scripts/format.sh
# Uses YAPF with Google style, 120-char column limit (see .style.yapf)
```

### Tests
```bash
pip install ".[test]"
pytest
```

## Architecture

### Training Loop (`verl/trainer/ppo/ray_trainer.py`)

`RayPPOTrainer` owns the main training loop and orchestrates Ray-based distributed workers:

1. **Rollout** — Actor generates trajectories (via vLLM)
2. **Reward** — `RewardManager` computes rule-based math scores
3. **Advantage** — `core_algos.py` computes advantages; the `algorithm.advantage` config key selects PSR/NSR/weighted
4. **Update** — PPO/GRPO gradient updates on Actor (and Critic for PPO)

Entry point `verl/trainer/main_ppo.py` parses Hydra config, instantiates Ray workers, and delegates to `RayPPOTrainer`.

### Worker Model (`verl/workers/`)

Workers are Ray actors assigned GPU resources via `RayResourcePool`. The three key roles:

- **ActorRolloutRefWorker** (`fsdp_workers.py`) — combines policy training (Actor), trajectory generation (Rollout via vLLM), and reference log-prob computation (Ref) in a single GPU-co-located worker
- **CriticWorker** — value function (PPO only)
- **RewardModelWorker** — optional learned reward model (most experiments use rule-based scoring instead)

Workers communicate via **DataProto** (`verl/protocol.py`): a named-tensor container that abstracts away sharding/device placement.

### Algorithm Selection (`algorithm.advantage`)

In `verl/trainer/ppo/core_algos.py`, advantage computation branches on this config key:
- `"positive"` → PSR: zero out advantages for incorrect samples
- `"negative"` → NSR: zero out advantages for correct samples  
- `"weighted"` → W-REINFORCE: blend via `positive_advantage_weight` (default 0.1)
- `"group_norm"` → GRPO-style group normalization

### Configuration (Hydra)

All hyperparameters flow through `verl/trainer/config/ppo_trainer.yaml`. Training shell scripts pass Hydra overrides like:
```
algorithm.advantage=negative
actor_rollout_ref.actor.ppo_mini_batch_size=256
```

OOM troubleshooting: reduce `actor_rollout_ref.actor.ppo_max_token_len_per_gpu`, `rollout.log_prob_max_token_len_per_gpu`, `ref.log_prob_max_token_len_per_gpu`.

### Reward Scoring (`verl/utils/reward_score/`)

`grader.py` implements rule-based math equality checking. `NaiveRewardManager` in `verl/workers/reward_manager/` calls it per-sample. PRIME-based scoring (code execution) is available as an alternative via config.

### Data Pipeline

Training data is parquet files loaded by `RLHFDataset` (`verl/utils/dataset/`). The dataset applies chat templates and tokenizes prompts. Eval datasets are pulled from HuggingFace (`TianHongZXY/MATH`, `TianHongZXY/AIME2025`, `TianHongZXY/amc23`).

---
> Source: [pedestrian6/nsr](https://github.com/pedestrian6/nsr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
