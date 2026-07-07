## persistworld

> PersistWorld is a Python 3.11 project. It is an action-conditioned generative world

# Repository Guidelines

## Project Structure & Module Organization
PersistWorld is a Python 3.11 project. It is an action-conditioned generative world
model for robot manipulation built on Stable Video Diffusion (SVD), with NFT
(No-Free-Training) post-training using visual quality rewards (LPIPS, PSNR, SSIM).

Project-wide defaults live in `config.py` via the `wm_args` dataclass; update it when
adding knobs so rollouts and training stay aligned.

| Path | Purpose |
|------|---------|
| `models/` | Core model code: `ctrl_world.py`, `pipeline_ctrl_world.py`, `unet_spatio_temporal_condition.py`, `ema.py`, `utils.py`, `action_adapter/` |
| `scripts/` | Entrypoints: `train_wm.py`, `train_nft_wm.py`, `eval_nft_wm.py`, `rollout_*.py`, utilities |
| `dataset/` | PyTorch Dataset class (`dataset_droid_exp33.py`) |
| `dataset_example/` | Latent-extraction scripts + small DROID sample subset and new-setup snapshots |
| `dataset_meta_info/` | `create_meta_info.py` — builds JSON manifests and action-stat files for training |
| `eval/` | Visual metric implementations: `metrics.py` (LPIPS, PSNR, SSIM) |
| `our_setup/` | Robot URDF/YAML config (`franka_panda.yaml`) for FK computations |
| `config.py` | `wm_args` dataclass — single source of truth for all hyperparameters |
| `muon.py` | Bundled Muon optimizer (not on PyPI); used only with `--optimizer muon` |

## Build, Test, and Development Commands
```bash
micromamba create -n persist-world python=3.11
micromamba activate persist-world
pip install -r requirements.txt
```

Always set `PYTHONPATH` to the repo root when running scripts:
```bash
export PYTHONPATH=/path/to/PersistWorld
```

**Replay subset (quick sanity check):**
```bash
PYTHONPATH=. CUDA_VISIBLE_DEVICES=0 python scripts/rollout_replay_traj.py \
    --dataset_root_path dataset_example \
    --dataset_meta_info_path dataset_meta_info \
    --dataset_names droid_subset \
    --svd_model_path ${SVD_PATH} \
    --clip_model_path ${CLIP_PATH} \
    --ckpt_path ${CKPT_PATH}
```

**Keyboard demo:**
```bash
PYTHONPATH=. CUDA_VISIBLE_DEVICES=0 python scripts/rollout_key_board.py \
    --task_type keyboard --keyboard lllrrr \
    --svd_model_path ${SVD_PATH} --clip_model_path ${CLIP_PATH} --ckpt_path ${CKPT_PATH}
```

**SV world model pre-training dry-run:**
```bash
PYTHONPATH=. WANDB_MODE=offline accelerate launch --main_process_port 29501 \
    scripts/train_wm.py --dataset_names droid_subset
```

**NFT post-training (single GPU):**
```bash
PYTHONPATH=. WANDB_MODE=offline CUDA_VISIBLE_DEVICES=0 python scripts/train_nft_wm.py \
    --dataset_root_path dataset_example \
    --dataset_meta_info_path dataset_meta_info \
    --dataset_names droid_subset \
    --use_lora --group_size 2
```

**Closed-loop eval:**
```bash
PYTHONPATH=. python scripts/eval_nft_wm.py \
    --dataset_root_path dataset_example --dataset_names droid_subset
```

## Rewards
Only visual quality rewards are used during NFT post-training:
- `--reward_w_lpips` (default 1.0) — LPIPS perceptual similarity (negated, lower is better)
- `--reward_w_ssim` (default 1.0) — SSIM structural similarity
- `--reward_w_psnr` (default 0.03125) — PSNR (scaled down to match magnitude)
- `--directly_combine_rewards` (default True) — combine reward values directly

## Coding Style & Naming Conventions
- 4-space indentation, Black-compatible formatting.
- Modules and functions: `snake_case`; classes: `CapWords`; CLI flags: hyphenated.
- Maintain the dataclass-config pattern in `config.py`; prefer explicit type hints.
- Document non-obvious tensor shapes inline (e.g. `# (B, T, C, H, W)`).

## Testing Guidelines
Validate changes with the replay subset to ensure rollouts stay numerically stable.
When touching NFT training code, run the single-GPU subset job and check that loss
values are finite. Add targeted assertions or lightweight pytest modules under
`models/tests/` if created.

## Commit & Pull Request Guidelines
Use short imperative subjects (`fix typo`, `add keyboard rollout`). Squash fixups
locally. PRs should state the scenario tested, GPU type, dataset variant, and link
any dependent checkpoints. Include before/after screenshots for visual rollouts and
mention config toggles that reviewers must replicate.

## Data & Checkpoint Management
Keep large checkpoints outside Git (reference via env vars or `wm_args` defaults).
Store secrets (W&B tokens, HuggingFace keys) in your shell — never commit them.
Large data directories excluded from Git:
- `dataset_example/droid/` (full pre-processed DROID, ~370 GB)
- `model_ckpt/`, `final_ckpts/`, `synthetic_traj/`, `eval_results/`

---
> Source: [Jai2500/PersistWorld](https://github.com/Jai2500/PersistWorld) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
