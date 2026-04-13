## deepreach-extended-to-no-dynamics

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DeepReach is a deep learning approach to high-dimensional Hamilton-Jacobi reachability analysis. It trains neural networks (SIREN architecture) to approximate value functions that solve HJ PDEs, enabling safety analysis of dynamical systems. Extended with **temporal consistency training** for learning value functions from trajectory data without explicit dynamics models.

The three primary systems are **CartPole** (4D), **Quadrotor2D** (6D), and **Quadrotor3D** (13D). All three learn from trajectory data and support temporal consistency training.

## Running Commands

### Environment Setup
```bash
python -m venv env && source env/bin/activate
pip install -r requirements.txt
# PyTorch must be installed separately for your CUDA version (commented out in requirements.txt)
```

### Training
```bash
# Config file (preferred)
python run_experiment.py -c configs/cartpole_50k_70k.cfg --mode train

# CLI arguments directly
python run_experiment.py --mode train --experiment_class DeepReach \
  --dynamics_class CartPole --data_root /path/to/data \
  --training_objective temporal_consistency \
  --experiment_name my_run --pretrain --pretrain_iters 50000 --num_epochs 120000

# Shell launcher scripts
./run_cartpole_50k_70k.sh
```

### Evaluation
```bash
python evaluation/eval_roa.py --experiment_dir runs/<name> --checkpoint model_final.pth \
  --test_set /path/to/test_labels.txt --t_eval 10.0
```

### Diagnostics
```bash
python scripts/inspect_pretrain_curriculum.py  # Inspect pretraining/curriculum batch generation and losses
```

### No test suite exists. Validation happens via `--mode test` (scenario optimization) and `eval_roa.py`.

## Three Primary Systems

### CartPole (4D)
- **State**: `[x, theta, x_dot, theta_dot]`
- **Dynamics class**: `CartPole` — has full analytical dynamics (`dsdt`, `hamiltonian`)
- **Dataset class**: `CartPoleDataset` — `angle_wrap_dims=[1]` (theta wrapping)
- **CLI params**: `u_max`, `x_bound`, `xdot_bound`, `thetadot_bound`, `gravity`, `cart_mass`, `pole_mass`, `pole_length`, `set_mode`, `data_root`, `load_physics_from_data_root`
- **Supports both** `hj_pde` and `temporal_consistency` training objectives
- **Physics loading**: Reads gravity, masses, length from `dataset_description.json` in `data_root` when `load_physics_from_data_root=True`
- **Boundary function**: L2 ball in normalized coordinates: `sqrt(sum((state/state_var)^2)) - 0.2`

### Quadrotor2D (6D)
- **State**: `[x, z, theta, x_dot, z_dot, theta_dot]`
- **Dynamics class**: `Quadrotor2D` — **no analytical dynamics** (`dsdt`/`hamiltonian` raise `NotImplementedError`)
- **Dataset class**: `Quadrotor2DDataset` — `angle_wrap_dims=[2]` (theta wrapping)
- **CLI params**: `x_bound`, `z_bound`, `theta_bound`, `xdot_bound`, `zdot_bound`, `thetadot_bound`, `set_mode`, `data_root`, `load_physics_from_data_root`
- **Supports only** `temporal_consistency` training (no dynamics model for PDE residual)
- **Boundary function**: Same L2 ball as CartPole

### Quadrotor3D (13D)
- **State**: `[x, y, z, qw, qx, qy, qz, vx, vy, vz, wx, wy, wz]` (quaternion attitude)
- **Dynamics class**: `Quadrotor3D` — **no analytical dynamics** (`dsdt`/`hamiltonian` raise `NotImplementedError`)
- **Dataset class**: `Quadrotor3DDataset` — `angle_wrap_dims=[]` (quaternions, no angle wrapping)
- **CLI params**: 13 individual bound params (`x_bound`, `y_bound`, ..., `wz_bound`), `set_mode`, `data_root`, `load_physics_from_data_root`
- **Supports only** `temporal_consistency` training
- **Special**: Reads `state_order` from `dataset_description.json` to handle flexible dimension ordering via `DEFAULT_STATE_ORDER` and `_BOUND_PARAM_MAP` class attributes
- **Boundary function**: Same L2 ball as CartPole

### Other Systems (legacy/reference)
Air3D, Dubins3D, Dubins4D, NarrowPassage, MultiVehicleCollision, RocketLanding, ReachAvoidRocketLanding, ParameterizedVertDrone2D, Quadrotor (13D full dynamics). These use `ReachabilityDataset` with synthetic sampling and `hj_pde` training.

## Architecture

### Dynamic Argument Discovery
`run_experiment.py` uses `inspect` to dynamically discover classes from `dynamics.dynamics` and `experiments.experiments` at runtime. Constructor parameters of dynamics/experiment classes are auto-exposed as CLI arguments. **Adding a new dynamics system requires zero changes to `run_experiment.py`** — just define the class inheriting from `dynamics.Dynamics`.

### Dataset Routing in `run_experiment.py`
```
dynamics_class == 'CartPole'    → CartPoleDataset   (angle_wrap_dims=[1])
dynamics_class == 'Quadrotor2D' → Quadrotor2DDataset (angle_wrap_dims=[2])
dynamics_class == 'Quadrotor3D' → Quadrotor3DDataset (angle_wrap_dims=[])
everything else                 → ReachabilityDataset (synthetic sampling)
```
All trajectory datasets inherit from `CartPoleDataset` — the differences are only `angle_wrap_dims` defaults.

### Module Structure
- **`dynamics/dynamics.py`** — Abstract `Dynamics` base class + 12 concrete systems. Defines state space, boundary conditions, Hamiltonian, optimal control, coordinate normalization (model-unit ↔ real-unit via `state_mean`/`state_var`), and value function interpretation.
- **`experiments/experiments.py`** — Abstract `Experiment` base class with single concrete `DeepReach` class. Contains the main training loop (~500 lines in `train()`), handling PDE training, temporal consistency, supervised loss, CSL phases, gradient adjustment, and checkpointing.
- **`utils/dataio.py`** — `ReachabilityDataset` (synthetic on-the-fly sampling), `CartPoleDataset` (trajectory file loading with RAM preloading, train/test splits, supervised labels, balanced sampling), `Quadrotor2DDataset`, `Quadrotor3DDataset` (both inherit CartPoleDataset, differ only in `angle_wrap_dims`).
- **`utils/modules.py`** — `SingleBVPNet` wrapping `FCBlock` with SIREN (sine) activation. MIT-licensed from Sitzmann.
- **`utils/losses.py`** — Loss factories: `init_brt_hjivi_loss`, `init_brat_hjivi_loss`, `init_temporal_consistency_loss`.
- **`utils/diff_operators.py`** — Batched Jacobian computation for value function gradients.
- **`utils/error_evaluators.py`** — Scenario optimization, validators (`ValueThresholdValidator`, `MLPValidator`), sample generators, and `target_fraction` computation.
- **`evaluation/eval_roa.py`** — ROA evaluation: loads test sets, computes model values, classifies safe/unsafe, reports metrics (precision, recall, F1), supports threshold optimization.

### No `__init__.py` files
Modules are imported with bare syntax (`from dynamics import dynamics`). Scripts must be run from the repo root.

## Training Flow

### Two Training Objectives
1. **`hj_pde`** (default): Standard HJ PDE residual minimization. Requires analytical dynamics (`dsdt`, `hamiltonian`). Works with CartPole and all legacy systems.
2. **`temporal_consistency`**: Uses observed state transitions from trajectory data. Loss: `max(dV/dt - ∇V·ẋ_obs, V - l(x))`. Works with CartPole, Quadrotor2D, Quadrotor3D. **Required** for Quad2D/Quad3D (no dynamics model).

### Loss Function Routing
```
training_objective == 'temporal_consistency' → init_temporal_consistency_loss()
training_objective == 'hj_pde' + loss_type == 'brt_hjivi' → init_brt_hjivi_loss()
training_objective == 'hj_pde' + loss_type == 'brat_hjivi' → init_brat_hjivi_loss()
```

### Phase 1: Pretraining (`--pretrain --pretrain_iters N`)
- All time samples pinned at `tMin`
- **HJ PDE mode**: Dirichlet loss only (match boundary function at t=0). With `exact` model: push raw output to 0.
- **Temporal consistency mode**: Push raw network output toward 0 (L1 loss on output)
- Runs for `pretrain_iters` epochs, then transitions to curriculum

### Phase 2: Curriculum Training (main)
- Time window linearly expands from `tMin` to `tMax` over `counter_start` → `counter_end` epochs
- `num_src_samples` always pinned at `t=tMin` for boundary anchoring
- **HJ PDE mode**: PDE residual `|dV/dt - H(x, ∇V)|` with optional BRT/BRAT max/min operators
- **Temporal consistency mode**: `max(dV/dt - ∇V·ẋ_obs, V - boundary_values)` (BRT operator)
- **Optional supervised loss** (`--supervised_weight > 0`): MSE between predicted V and labels from `roa_labels.txt` or `cal_set.txt`. Supports balanced sampling and class-specific weights (`supervised_safe_weight`, `supervised_unsafe_weight`).
- **Gradient adjustment** (`--adj_rel_grads`, HJ PDE only): Scales Dirichlet loss gradient to match PDE gradient magnitude (EMA alpha=0.1)

### Phase 3: CSL — Cost-Supervised Learning (optional, HJ PDE only)
- Triggered periodically: every `epochs_til_CSL` epochs after pretrain ends
- Runs `scenario_optimization()` to roll out trajectories under current optimal control, generating cost labels
- Inner training loop with separate optimizer (`CSL_lr`):
  - MSE on trajectory costs (weighted by `CSL_loss_weight`) + PDE residual
  - Early stopping when `val_loss < CSL_loss_frac_cutoff * initial_val_loss`
- Logs recovered safe-set fractions to WandB

### Trajectory Data Format
```
data_root/
├── dataset_description.json    # Physics params, achieved_bounds, state_order (Quad3D)
├── trajectories/
│   ├── sequence_0000.txt       # Tab-separated: each row = state at timestep
│   ├── sequence_0001.txt
│   └── ...
├── roa_labels.txt              # 5-col (state, label) or cal_set.txt 9-col (state, next_state, label)
└── shuffled_indices/
    └── shuffled_indices.txt    # Train/test split indices
```

- `CartPoleDataset` loads trajectories, pads short ones, supports RAM preloading
- **HJ PDE sampling** (`_sample_hj_batch`): Random states from random trajectories, times from curriculum window
- **Temporal consistency sampling** (`_sample_temporal_batch`): One-step transitions, computes `obs_flow = (next_state - curr_state) / dt` with angle wrapping

### Value Function Interpretation (`--deepreach_model`)
- **`exact`** (default for all 3 systems): `V = output * t * var/normto + l(x)` — forces exact boundary at t=0
- **`diff`**: `V = output * var/normto + l(x)` — learns difference from boundary
- **`vanilla`**: `V = output * var/normto + mean` — standard approach

## Configuration

Uses `ConfigArgParse` — arguments can come from CLI, config files (`-c configs/*.cfg`), or environment variables. Config files in `configs/` are key=value format.

### Available Configs
| Config | System | Epochs | Key Settings |
|--------|--------|--------|-------------|
| `cartpole_50k_70k.cfg` | CartPole | 120k (50k pretrain + 70k curriculum) | Train split, RAM, uniform sampling, cal_set supervised, 2x96 SIREN |
| `cartpole_large_X_calset.cfg` | CartPole | 90k (50k pretrain) | All trajectories, uniform sampling |
| `cartpole_large_X_calset_15k_6k.cfg` | CartPole | 15k (6k pretrain) | Short run variant |
| `cartpole_large_X_calset_15k_6k_trainsplit_ram.cfg` | CartPole | 15k (6k pretrain) | Train split + RAM |
| `paper_air3d.cfg` | Air3D | 100k (10k pretrain) | 3x512, synthetic |
| `paper_9d_multivehicle.cfg` | MultiVehicleCollision | 100k (60k pretrain) | 3x512, synthetic |
| `paper_10d_narrow_passage.cfg` | NarrowPassage | 100k (60k pretrain) | 3x512, BRAT |

### Output Structure
```
runs/<experiment_name>/
├── orig_opt.pickle              # Serialized training options
├── run_command.txt / .sh        # Exact command that was run
├── training/
│   ├── summaries/               # TensorBoard event files
│   └── checkpoints/
│       ├── model_current.pth    # Latest checkpoint (state_dict only)
│       ├── model_epoch_NNNN.pth # Periodic (model + optimizer state)
│       └── model_final.pth      # Final model
```

## Adding a New Trajectory-Based System

1. Add a dynamics class in `dynamics/dynamics.py` inheriting from `Dynamics`
   - For temporal-consistency-only systems: `dsdt`/`hamiltonian` can raise `NotImplementedError`
   - Implement: `state_test_range`, `equivalent_wrapped_state`, `boundary_fn`, `sample_target_state`, `cost_fn`, `optimal_control`, `optimal_disturbance`, `plot_config`
   - Set `state_mean`/`state_var` from bounds (optionally loaded from `dataset_description.json`)
   - Constructor params become CLI arguments automatically
2. Add a dataset class in `utils/dataio.py` inheriting from `CartPoleDataset`
   - Only override needed: `angle_wrap_dims` default (which dimensions have angular wrapping)
3. Add routing in `run_experiment.py` (~line 232): add your class name to the `if dynamics_class in (...)` block with the matching dataset class
4. Create a config file in `configs/` and optionally a shell launcher script

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sandeepKBM)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sandeepKBM)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
