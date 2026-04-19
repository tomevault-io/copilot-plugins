## so-101-rl

> Train RL policies in MuJoCo to make the SO-101 arm perform basic manipulation tasks.

# MuJoCo RL Training — SO-101

## Goal

Train RL policies in MuJoCo to make the SO-101 arm perform basic manipulation tasks.

## Model

- Scene: `so101/scene.xml` (includes `so101.xml` + floor/lighting)
- Assets: `so101/assets/` (STL meshes)
- Source: [MuJoCo Menagerie](https://github.com/google-deepmind/mujoco_menagerie/tree/main/robotstudio_so101)
- Requires MuJoCo >= 3.1.3 (installed: 3.5.0)

## Tasks

### Shared

- `base_env.py` — Base Gymnasium env with MuJoCo setup, contact detection, rendering, reset/step skeleton (20-dim obs, 6-dim action)

### Lift

Grasp a cube and lift it to 10cm. Simpler grasping prerequisite for pick-and-place.

- `so101/scene_lift.xml` — scene with cube only
- `lift_env.py` — Gymnasium env (height-progress reward)
- `conf/env/lift.yaml` — env config

### Pick and Place

Pick up a cube and place it at a target location. 3-phase task: REACH → PLACE → RETURN.

- `so101/scene_pickplace.xml` — scene with free-body cube, place target, and ring
- `pickplace_env.py` — Gymnasium env (phase-based reward)
- `conf/env/pickplace.yaml` — env config

## Training

- `train.py` — Training with Hydra config + W&B logging
- `conf/config.yaml` — shared hyperparameters (train, algorithm, wandb), default env: pickplace
- `conf/env/` — per-env config group (selected via `env=lift`, `env=pickplace`, or `env=multitask`)

### Usage

All commands must run in the `mujoco_env` conda environment:

```bash
conda activate mujoco_env

python train.py env=lift                     # train lift
python train.py env=pickplace                # train pick-and-place (default)
python train.py env=multitask                # train on both lift + pickplace (50/50)
python train.py env=lift wandb.enabled=false # without W&B
python train.py train.total_timesteps=200000 # override params
python eval.py env=lift                      # eval lift checkpoint
python eval.py env=pickplace                 # eval pickplace checkpoint
python eval.py model=best                    # eval best model
python eval.py model=path/to/model.zip       # eval specific model
python show_starts.py                        # visualize spawn positions
```

### Stack

- **Conda env:** `mujoco_env`
- **Config:** Hydra (`conf/config.yaml` + per-env overrides)
- **Logging:** W&B (entity: `mvlasenko`, project: `robot-arm`)
- **Algorithm:** SAC or PPO (Stable-Baselines3)
- **Deps:** gymnasium, stable-baselines3, wandb, hydra-core

## Coding Principles

- **Fail fast, fail loud.** No blanket `try/except`, no `except Exception`, no swallowing errors. If something breaks, let it crash with a clear traceback. Only catch specific exceptions when there's a real recovery path.
- **No magic.** No `getattr`/`setattr` with string keys, no `**kwargs` passthrough when explicit args work, no dynamic dispatch when a simple `if`/`dict` suffices. Code should be readable without running it.
- **Single source of truth.** Don't duplicate constants, config values, or defaults across files. One place defines it, everywhere else reads from there. `conf/config.yaml` owns all hyperparameters.
- **No broken intermediate states.** Don't leave code half-working. If a change touches multiple files, all files must be consistent before moving on. Tests/imports should pass at every step.
- **Explicit over defensive.** Require values instead of falling back to defaults silently (`cfg["key"]` not `cfg.get("key", default)`). If a required value is missing, that's a bug — surface it immediately.
- **No dead code.** Delete unused variables, imports, and functions. Don't comment things out "for later." Version control exists.
- **No redundant comments.** Don't restate what the code already says. Comments explain *why*, not *what*.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mikhail-vlasenko) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
