## scsi

> Research codebase for **Self-Consistent Stochastic Interpolants (SCSI)**: recovering a prior over clean signals from corrupted observations via an EM loop where the E-step trains a stochastic-interpolant velocity field and the M-step updates the prior. Four parallel experiment scopes scale up complexity (toy 2D → toy 3D → MNIST → ModelNet cryoEM); they all share the same module skeleton, so learn one and you know them all.

# scsi — navigation guide for Codex

Research codebase for **Self-Consistent Stochastic Interpolants (SCSI)**: recovering a prior over clean signals from corrupted observations via an EM loop where the E-step trains a stochastic-interpolant velocity field and the M-step updates the prior. Four parallel experiment scopes scale up complexity (toy 2D → toy 3D → MNIST → ModelNet cryoEM); they all share the same module skeleton, so learn one and you know them all.

## Environment

- Python **3.13** (`.python-version`), managed by **uv** (`pyproject.toml`, `uv.lock`).
- Platform-specific torch wheels configured in `pyproject.toml`: CPU on darwin, CUDA 13.0 on linux.
- Key deps: `torch`, `diffusers` (DiT, UNet3D), `wandb` (optional), `trimesh` / `scikit-image` (3D), `torch-geometric`.
- Always run via `uv run python <script>` — never bare `python`, the torch wheel resolution depends on uv.

## Repo map

| Path | What's there |
|---|---|
| `image_2d/` | MNIST reconstruction under AWGN / MRA (multi-reference alignment) corruption, conditional DiT. |
| `simple_3d/` | ModelNet10 voxel reconstruction under SO(3) → Radon → AWGN (cryoEM forward model), UNet3D. |
| `toy_2d/` | Two-moons exploration. Two notebooks + one reverse-dream script. |
| `toy_3d/` | Synthetic primitives (sphere / cube / cylinder / ellipsoid / torus) under cryoEM corruption. |
| `toy_2d_pc/` | 2D point-cloud SCSI: MNIST-digit silhouettes under a CryoEM-style 2D-image→1D-sinogram channel (gaussian/disk/histogram rendering + SO(2) tilt series). Self-contained package, see `toy_2d_pc/AGENTS.md`. |
| `data/` | Auto-downloaded datasets (currently just MNIST). |
| `writing/` | Reserved for papers — currently empty. |
| `main.py` | Top-level stub (`print("Hello from scsi!")`). Ignore. |
| `pyproject.toml` | Single source of truth for deps + torch index config. |

## Shared experiment skeleton

Each of `image_2d/`, `simple_3d/` (and to a lesser extent `toy_3d/`) follows the same layout:

| File | Role |
|---|---|
| `si.py` | Stochastic interpolant. Schedules (`linear`, `GVP`), `interpolant()`, `loss_func()`, `sample()`. |
| `corruption.py` | Forward channel: `forward_channel(x, noise_std, ...)`. AWGN / MRA in 2D; SO(3)+Radon+AWGN in 3D. |
| `model.py` | Conditional velocity net (DiT for 2D, UNet3D for 3D). Input is the channel-concat of `(x_t, observation)`. |
| `data.py` | Dataset loader. Inputs normalized to `[-1, 1]`. |
| `em.py` | EM loop: `train_estep`, `update_prior`, `log_em_step_wandb`. |
| `main.py` | argparse CLI that wires everything together and runs the EM loop. |

`toy_3d/` flattens this into two top-level scripts (`train.py` supervised, `scsi_train.py` unsupervised SCSI). `toy_2d/` is notebook-driven.

## How to run

```bash
# Image MNIST
uv run python image_2d/main.py
uv run python image_2d/main_warmup_curriculum.py   # curriculum variant
uv run python image_2d/study_coupling.py           # coupling-fraction ablation

# 3D ModelNet cryoEM
uv run python simple_3d/main.py

# Toy 3D (synthetic shapes)
uv run python toy_3d/train.py        # supervised baseline
uv run python toy_3d/scsi_train.py   # unsupervised SCSI from 2D projections only

# Toy 2D (two-moons)
uv run python toy_2d/lifted_2dtoys_reverse_dream.py
# or open toy_2d/lifted_2dtoys.ipynb / lifted_2dtoys_cm.ipynb

# Cluster: SLURM scripts live next to the entry points
sbatch image_2d/submit_awgn_mnist_transformer.SBATCH
sbatch image_2d/submit_mra_mnist_transformer.SBATCH
sbatch image_2d/submit_drop_mra_mnist_transformer.SBATCH
sbatch image_2d/submit_curriculum_mra_multiem.SBATCH
sbatch simple_3d/submit_cryoem_modelnet.SBATCH
```

All entry points take CLI flags via argparse (corruption type, dataset size, EM steps, etc.). Read the top of each `main.py` for the full list.

## Conventions

- `INTEGRATION_SCALE = 999` is sprinkled across the codebase — it's the diffusers timestep-embedding range; pass `t * INTEGRATION_SCALE` when calling the velocity network.
- Grid size: `IMAGE_SIZE = 32` (`image_2d`), `VOL_SIZE = 32` (`simple_3d`, `toy_3d`).
- Device autodetect: CUDA → MPS → CPU at every entry point.
- W&B is optional; logging degrades gracefully when not configured.
- Gitignored as ephemeral: `*.png`, `*.mp4`, `*.gif`, `*.pt`, `checkpoints*`, `priors*`, `wandb/`.

## Gotchas

- **The four experiment dirs duplicate near-identical code with small divergences.** Fixing a bug in `image_2d/si.py` probably means fixing `simple_3d/si.py` and `toy_3d/`'s inlined version too. Always grep across all four before assuming a change is local.
- `image_2d/archive/` and `image_2d/results/` exist — confirm whether a file is the live entry point or a snapshot before editing.
- `toy_2d/*.ipynb` carry large embedded cell outputs (1.5–5.6 MB). Avoid re-executing for trivial reasons; the diff bloat is real.
- `README.md` is a one-liner — this file is the actual docs. Update it when the module layout changes.

---
> Source: [clarkmiyamoto/scsi](https://github.com/clarkmiyamoto/scsi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
