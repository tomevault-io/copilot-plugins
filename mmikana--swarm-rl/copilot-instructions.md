## swarm-rl

> This repository provides a codebase for training reinforcement learning policies for quadrotor swarms. It integrates a flight dynamics simulator, forked and extended from `gym_art` to support quadrotor swarms, with the Sample Factory framework for training control policies. The project focuses on decentralized control, collision avoidance, and navigation for multi-quadrotor systems, with ongoing research into efficient policy deployment and refinement.

# Project: QuadSwarm-RL

## Project Overview
This repository provides a codebase for training reinforcement learning policies for quadrotor swarms. It integrates a flight dynamics simulator, forked and extended from `gym_art` to support quadrotor swarms, with the Sample Factory framework for training control policies. The project focuses on decentralized control, collision avoidance, and navigation for multi-quadrotor systems, with ongoing research into efficient policy deployment and refinement.

## Technologies Used
*   Python
*   PyTorch (implied by Sample Factory and RL context)
*   Numba (for performance in `gym_art.quadrotor_multi`)
*   Sample Factory (for reinforcement learning training)
*   Weights & Biases (WandB) for experiment monitoring

## Building and Running

### Installation
To set up the environment, it is recommended to use `conda` with Python versions >=3.11:

```bash
conda create -n swarm-rl python=3.11
conda activate swarm-rl
git clone https://github.com/Zhehui-Huang/quad-swarm-rl.git
cd quad-swarm-rl
pip install -e .
```
This command installs all necessary dependencies, including PyTorch.

### Training
Experiments can be initiated using either a shell script or Python runner scripts:

*   **Using `train.sh` (baseline experiment):**
    ```bash
    bash train.sh
    ```
    Adjust the number of workers in the script to match your machine's logical CPU cores.

*   **Using runner scripts (e.g., `swarm_rl.runs.single_quad.single_quad`):**
    ```bash
    python -m sample_factory.launcher.run --run=swarm_rl.runs.single_quad.single_quad --max_parallel=4 --pause_between=1 --experiments_per_gpu=1 --num_gpus=4
    ```
    Modify parameters (`--max_parallel`, `--num_gpus`) according to your hardware setup.

#### Weights & Biases (WandB) Support
To enable WandB for experiment monitoring:
1.  Log in to WandB locally: `wandb login`
2.  Add `--with_wandb=True` to your training command, along with optional parameters like `--wandb_user`, `--wandb_project`, `--wandb_group`, and `--wandb_tags`.

### Testing Trained Models
To test a trained model:
```bash
python -m swarm_rl.enjoy --algo=APPO --env=quadrotor_multi --replay_buffer_sample_prob=0 --quads_use_numba=False --quads_render=True --train_dir=PATH_TO_TRAIN_DIR --experiment=EXPERIMENT_NAME --quads_view_mode CAMERA_VIEWS
```
Replace `PATH_TO_TRAIN_DIR` and `EXPERIMENT_NAME` with values found in your trained model's `cfg.json` file. `CAMERA_VIEWS` can be a space-separated list of `[topdown, global, chase, side, corner0, corner1, corner2, corner3, topdownfollow]`.

### Unit Tests
To run the project's unit tests:
```bash
./run_tests.sh
```

## Development Conventions
*   **Code Structure**: The project is organized into `gym_art` (simulator) and `swarm_rl` (RL training and related utilities).
*   **Configuration**: Experiment parameters are often defined in Python runner scripts within `swarm_rl/runs/`.
*   **Version Control**: Standard Git workflow for contributions (implied by `.git` directory and common practice). Github issues and pull requests are welcome.
*   **Documentation**: The `README.md` file is comprehensive for setup and basic usage.

---
> Source: [mmikana/swarm-rl](https://github.com/mmikana/swarm-rl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
