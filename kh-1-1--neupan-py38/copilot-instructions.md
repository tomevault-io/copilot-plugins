## neupan-py38

> **NeuPAN** (Neural Proximal Alternating-minimization Network) is a Python-based robotic motion planning framework. It implements an **end-to-end**, **real-time**, and **map-free** navigation algorithm based on Model Predictive Control (MPC).

# NeuPAN Project Context

## Project Overview
**NeuPAN** (Neural Proximal Alternating-minimization Network) is a Python-based robotic motion planning framework. It implements an **end-to-end**, **real-time**, and **map-free** navigation algorithm based on Model Predictive Control (MPC).

The core innovation lies in integrating deep learning (via a DUNE model) with mathematical optimization to map raw obstacle point data directly to robot control actions, enabling navigation in cluttered, unknown environments without explicit mapping.

**Key Metadata:**
- **Paper:** Accepted by IEEE Transactions on Robotics (T-RO 2025).
- **Authors:** Han Ruihua et al. (HKU).
- **Supported Robots:** Differential drive (`diff`) and Ackermann steering (`acker`).

## Architecture & Structure
- **`neupan/`**: Core library source code.
    - `neupan.py`: Main entry point/class.
    - `robot/`: Robot kinematics and dynamics definitions.
    - `configuration/`: Configuration handling.
- **`example/`**: Usage examples and experiments.
    - `run_exp.py`: Main script to run simulations.
    - `dune_train/`: Scripts for training the DUNE model (custom robot geometries).
- **`baseline_methods/`**: Comparative methods used in the research.
- **`docs/`**: Documentation and analysis reports.

## Installation & Setup
The project requires Python 3.8+ (specifically tested on 3.8 and 3.10).

1. **Install NeuPAN:**
   ```bash
   pip install -e .
   ```
2. **Install Simulation Environment (Required for examples):**
   ```bash
   pip install ir-sim
   ```

## Key Commands
**Running Examples (Simulation):**
Located in the `example/` directory. Use `run_exp.py` with experiment (`-e`) and driver (`-d`) flags.

```bash
# Convex obstacles with Ackermann robot
python example/run_exp.py -e convex_obs -d acker

# Dynamic obstacles with Differential drive robot
python example/run_exp.py -e dyna_obs -d diff -v
```

**Training DUNE Model:**
If changing robot geometry, retraining is required:
```bash
# See example/dune_train/ folder
python example/dune_train/acker_flex_pdhg_no-projection_robot.py
```

## Configuration
The system relies heavily on YAML configuration.
- Global defaults and parameters for MPC (receding horizon, steps), Robot (kinematics, shape), and Training are typically found in YAML files or the `configuration` module.

## Development Notes
- **Dependencies:** Relies on `torch`, `cvxpy`, `cvxpylayers`, `ecos` for optimization and learning.
- **Hardware:** CPU is recommended for the solver (cvxpy), while GPU is used for training the DUNE model.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kh-1-1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kh-1-1)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
