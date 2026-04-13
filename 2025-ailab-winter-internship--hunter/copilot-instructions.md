## hunter

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

All `colcon` commands must be run from the workspace root `/home/ubuntu/ros_ws` (not from `src/`).

```bash
# Source ROS2 Humble
source /opt/ros/humble/setup.bash

# Build all packages
colcon build --metas src/colcon.meta --symlink-install

# Build a single package
colcon build --packages-select hunter_planning

# Build with tests enabled
colcon build --packages-select hunter_planning --cmake-args -DBUILD_TESTING=ON

# Run all tests
colcon test --packages-select hunter_planning
colcon test-result --verbose

# Run a single test binary directly (after building with tests)
./build/hunter_planning/test_dubins_curve
# All test binaries: test_global_map, test_dijkstra, test_dubins_curve, test_hybrid_astar,
#   test_path_smoother, test_path_resampler, test_velocity_profiler,
#   test_state_lattice, test_trajectory_generator, test_velocity_planner

# Source install and launch
source install/setup.bash
ros2 launch hunter_planning planner_controller_node.launch.py

# Test node launches (with RViz)
ros2 launch hunter_planning test_global.launch.py
ros2 launch hunter_planning test_local.launch.py

# Simulation
ros2 launch hunter2_gazebo gazebo.launch.py
```

## Workspace Package Overview

| Package               | Purpose                                                                                                     |
| --------------------- | ----------------------------------------------------------------------------------------------------------- |
| `hunter_msgs2`        | Custom message definitions: `EgoState`, `TrajectoryPath`, `TrajectoryPoint`, `Risk`                         |
| `hunter_planning`     | Core planning system (global + local planners) — see [hunter_planning/CLAUDE.md](hunter_planning/CLAUDE.md) |
| `hunter_interface`    | CAN-bus bridge between ROS2 and physical Hunter robot hardware                                              |
| `hunter2_gazebo`      | Gazebo simulation worlds and launch files                                                                   |
| `hunter2_description` | URDF/Xacro robot model and meshes                                                                           |
| `hunter2_bringup`     | Top-level launch configurations                                                                             |

## High-Level Architecture

### Data Flow

```
/goal_pose (RViz) ──────────────────────────────────────────────────────┐
/global_costmap/costmap ──────────────────────────────────────────────→ planner_controller_node
/local_costmap/costmap  ──────────────────────────────────────────────→  (hunter_planning)
/ego_state (EgoState)   ──────────────────────────────────────────────→  │
/safety/risk_status     ──────────────────────────────────────────────→  │
                                                                          │
                        /planner/global_path (nav_msgs/Path) ←───────────┤
                        /planner/local_path  (TrajectoryPath) ←──────────┤
                                │
                                ↓ (cmd_vel extracted from trajectory)
                        hunter_interface_node ──(CAN bus)──→ Hunter Robot
                                │
                                ↓ (odometry)
                        /odom (nav_msgs/Odometry)
```

### Planner State Machine

`planner_controller_node` runs at 20 Hz with states:
`IDLE → GLOBAL_PLANNING → LOCAL_PLANNING ↔ STOP_MODE → EMERGENCY_STOP`

A new `/goal_pose` triggers global re-planning. The local planner runs continuously during `LOCAL_PLANNING`, reacting to updated costmaps and risk scores.

### Hunter Interface Node

Converts `/cmd_vel` (linear + angular velocity) to Ackermann steering via `tan(δ) = (L × ω) / v`. Stops the robot if no command is received for 0.2 s. Runs at 50 Hz.

## Custom Messages (`hunter_msgs2`)

- **`EgoState`**: `x, y, yaw, v, a, yaw_rate` — current robot pose and kinematics
- **`TrajectoryPoint`**: pose + `longitudinal_velocity`, `longitudinal_acceleration`, `curvature`, `time_from_start`
- **`TrajectoryPath`**: array of `TrajectoryPoint`
- **`Risk`**: `score` (float32, 0.0–1.0)

## Vehicle Parameters

`hunter_planning/config/vehicle_config.yaml` is the **single source of truth** for all vehicle geometry and dynamics limits (max velocity 1.5 m/s, max decel 2.0 m/s², 3-circle collision footprint). All planners load from this file.

## Code Conventions

- Code comments are in **Korean**; identifiers and APIs are in **English**
- Logging uses **spdlog** (not `RCLCPP_INFO`)
- Linear algebra via **Eigen3**; configuration via **yaml-cpp**
- C++17 required (`std::optional`, structured bindings, etc.)
- Tests use **GTest** via `ament_cmake_gtest`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/2025-AILAB-WINTER-INTERNSHIP)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/2025-AILAB-WINTER-INTERNSHIP)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
