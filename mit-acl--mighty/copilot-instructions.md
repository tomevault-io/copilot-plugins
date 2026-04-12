## mighty

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MIGHTY (Hermite Spline-based Efficient Trajectory Planning) is a ROS 2 package for autonomous robot trajectory planning. It supports UAVs and ground robots (Pioneer 3-AT) in both simulation and hardware. Developed at MIT ACL by Kota Kondo.

## Build Commands

```bash
# Build the workspace (from mighty_ws root)
cd ~/code/mighty_ws
colcon build

# Build only the mighty package
colcon build --packages-select mighty

# Source after building
source install/setup.bash
```

The build system is **ament_cmake** (ROS 2 Humble on Ubuntu 22.04). CMake flags include `-Wall -Wextra -Wpedantic` with OpenMP enabled.

## Running Simulations

All simulation modes use `scripts/run_sim.py` which generates tmuxp YAML configs:

```bash
cd ~/code/mighty_ws

# Multi-agent simulation (10 agents, fake sensing)
python3 src/mighty/scripts/run_sim.py --mode multiagent

# Single-agent Gazebo UAV simulation
python3 src/mighty/scripts/run_sim.py --mode gazebo

# Ground robot (Pioneer 3-AT) Gazebo simulation
python3 src/mighty/scripts/run_sim.py --mode gazebo --ground-robot

# Custom goal, environment, agent count
python3 src/mighty/scripts/run_sim.py --mode gazebo --env easy_forest --goal 100 50 3
python3 src/mighty/scripts/run_sim.py --mode multiagent --num-agents 5

# Preview generated YAML without launching
python3 src/mighty/scripts/run_sim.py --mode gazebo --dry-run
```

Docker alternative (from `docker/` directory): `make run`, `make run-gazebo`, `make run-multiagent NUM_AGENTS=5`.

## Testing

```bash
# L-BFGS solver unit tests
ros2 run mighty test_lbfgs_solver
```

Jupyter notebooks in `scripts/` contain optimization tests (`test_unconstrained_opt.ipynb`, etc.).

## Architecture

### Core Pipeline

1. **MightyNode** (`src/mighty/mighty_node.cpp`, `include/mighty/mighty_node.hpp`) — Main ROS 2 node. Manages subscriptions (point clouds, odometry, Vicon, inter-agent trajectories), replanning timer callbacks, and trajectory publishing. Orchestrates all other components.

2. **Mighty** (`src/mighty/mighty.cpp`, `include/mighty/mighty.hpp`) — Core planning algorithm. Implements Hermite spline-based trajectory generation with state machine: YAWING → TRAVELING → GOAL_SEEN → GOAL_REACHED.

3. **L-BFGS Solver** (`src/mighty/lbfgs_solver.cpp`) — Local trajectory optimizer. Optimizes Hermite polynomial splines subject to constraints (velocity, acceleration, jerk, bodyrate, tilt, thrust) and costs (obstacle avoidance, smoothness, time).

4. **DGP (Discrete Global Planner):**
   - **DGPManager** (`src/dgp/dgp_manager.cpp`) — Manages occupancy maps, obstacle inflation, and convex decomposition. Interfaces between sensor data and planning.
   - **DGPPlanner** (`src/dgp/dgp_planner.cpp`) — Abstract planner base supporting JPS (Jump Point Search) and A* with heat maps.
   - **GraphSearch** (`src/dgp/graph_search.cpp`) — Core graph search with collision checking and path post-processing.

### Supporting Nodes

- **FakeSim** (`src/mighty/fake_sim.cpp`) — Lightweight multi-agent simulator that distributes simulated sensor data without Gazebo.
- **ObstacleTrackerNode** (`src/mighty/obstacle_tracker_node.cpp`) — Extracts dynamic obstacles from point clouds using PCL.
- **PurePursuit** (`src/mighty/pure_pursuit.cpp`) — Path-following controller for ground robots.
- **Coordinate converters** — `convert_odom_to_state`, `convert_vicon_to_state`, `convert_velodyne_to_ros_time`.

### Configuration

- `config/mighty.yaml` — Primary config (~150 parameters): planner bounds, dynamics constraints, cost weights, safety thresholds, visualization levels.
- `config/mighty_ground_robot.yaml` — Ground robot (Pioneer 3-AT) overrides.
- `config/hw_mighty.yaml` / `hw_mighty_rover.yaml` — Hardware-specific configs.
- `include/mighty/mighty_type.hpp` — Parameter struct definition (all configurable parameters declared here).

### Launch System

- `launch/base_mighty.launch.py` — Starts Gazebo world + optional RViz.
- `launch/onboard_mighty.launch.py` — Main node launcher. Handles UAV/ground robot modes, sensor configs (RealSense, Livox), localization (DLIO vs Vicon), frame alignment for multi-agent.
- `launch/simulator.launch.py` — FakeSim setup for multi-agent mode.

### Key Dependencies

- **Eigen3** — Linear algebra throughout the planner.
- **PCL** — Point cloud processing for obstacle detection.
- **DecompROS2** — Convex decomposition for safe flight corridors.
- **dynus_interfaces** — Custom ROS 2 message types for trajectories and dynamics.
- **acl-mapping** (`global_mapper_ros`) — 3D occupancy mapping.
- **uav_simulator** — Quadrotor physics simulation in Gazebo.

All external dependencies are version-pinned in `mighty.repos` and installed via `setup.sh`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mit-acl)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mit-acl)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
