## trackdlo2

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TrackDLO2 is a ROS2 Humble + Gazebo Fortress system for real-time tracking and manipulation of Deformable Linear Objects (DLO). It uses RGB-D camera input (RealSense D435 or Gazebo simulation) to detect/track DLO via CPD-LLE algorithm, with UR5 robot arm for autonomous manipulation.

## Build & Development Commands

### Native Build (ROS2 workspace)
```bash
# Build all packages
colcon build --packages-select trackdlo_msgs trackdlo_perception trackdlo_utils trackdlo_bringup trackdlo_description --cmake-args -DCMAKE_BUILD_TYPE=Release

# Build single package
colcon build --packages-select trackdlo_perception --cmake-args -DCMAKE_BUILD_TYPE=Release

# Source after build
source install/setup.bash
```

### Docker Build & Run
```bash
cd docker/
bash build.sh              # All images
bash build.sh sim          # Simulation only
bash build.sh realsense    # RealSense only

# Run (GPU type: amd | nvidia)
./run.sh amd sim           # Simulation
./run.sh nvidia realsense  # RealSense
./run.sh amd sim -d        # Detached mode
```

### Tests & Linting
```bash
# Run tests
colcon test --packages-select trackdlo_perception trackdlo_utils trackdlo_bringup --return-code-on-test-failure

# Lint Python
flake8 trackdlo_perception/trackdlo_perception/ trackdlo_utils/trackdlo_utils/ --max-line-length=150 --ignore=E501,W503,E741
```

### Launch Commands
```bash
# Simulation full pipeline
ros2 launch trackdlo_bringup full_pipeline.launch.py

# Perception only (sim)
ros2 launch trackdlo_bringup trackdlo.launch.py

# RealSense test
ros2 launch trackdlo_bringup realsense_test.launch.py
ros2 launch trackdlo_bringup realsense_test.launch.py segmentation:=hsv_tuner
ros2 launch trackdlo_bringup realsense_test.launch.py segmentation:=sam2
```

## Architecture

### Multi-Container Docker Architecture
Four containers communicating via ROS2 topics/services only (host network, CycloneDDS):
- **trackdlo-gazebo**: Gazebo simulation, robot_state_publisher, controllers
- **trackdlo-perception**: init_tracker (Python) + trackdlo C++ tracking node
- **trackdlo-moveit**: move_group (OMPL) + dlo_manipulation_node (state machine)
- **trackdlo-viz**: RViz2 visualization

### Package Structure

| Package | Language | Build | Role |
|---------|----------|-------|------|
| `trackdlo_perception` | C++17 + Python | ament_cmake | Core CPD-LLE tracking algorithm + initialization |
| `trackdlo_moveit` | C++17 | ament_cmake | MoveIt2 planning + DLO manipulation state machine |
| `trackdlo_bringup` | Launch/Config | ament_cmake | Launch files, YAML params, RViz config, Gazebo worlds |
| `trackdlo_description` | URDF/xacro | ament_cmake | UR5 + RealSense D435 robot description |
| `trackdlo_utils` | Python | ament_python | Depth converter, HSV tuner, SAM2 segmentation, test tools |
| `trackdlo_msgs` | ROS IDL | ament_cmake | Custom messages (minimal, for future use) |

### Processing Pipeline
1. **Initialization** (`trackdlo_perception/trackdlo_perception/initialize.py`): HSV threshold → skeleton extraction → spline fitting → 45 equally-spaced 3D nodes → publishes once to `/trackdlo/init_nodes`
2. **Per-frame tracking** (`trackdlo_perception/src/trackdlo_node.cpp` + `trackdlo.cpp`): RGB-D sync → HSV mask → point cloud → voxel downsample → visibility estimation → CPD-LLE EM iterations → updated node positions
3. **Robot manipulation** (`trackdlo_moveit/src/dlo_manipulation_node.cpp`): Subscribes to tracking results → extracts endpoints → state machine (GOTO_A ↔ GOTO_B) → MoveIt trajectory planning → execution

### Key Source Files
- `trackdlo_perception/src/trackdlo.cpp` — CPD-LLE algorithm core (cpd_lle method)
- `trackdlo_perception/src/trackdlo_node.cpp` — ROS2 node: image sync, preprocessing, visibility detection
- `trackdlo_perception/src/utils.cpp` — Projection, depth conversion, occlusion helpers
- `trackdlo_perception/trackdlo_perception/initialize.py` — Initialization pipeline (InitTrackerNode)
- `trackdlo_moveit/src/dlo_manipulation_node.cpp` — UR5 manipulation state machine
- `trackdlo_bringup/config/trackdlo_params.yaml` — Simulation tracking parameters
- `trackdlo_bringup/config/realsense_params.yaml` — RealSense tracking parameters

### Key ROS2 Topics
- `/trackdlo/init_nodes` (PointCloud2) — Initial 45 nodes, published once
- `/trackdlo/results_pc` (PointCloud2) — Per-frame tracked node positions
- `/trackdlo/segmentation_mask` (Image) — Optional external mask from HSV tuner or SAM2
- `/trackdlo/endpoints` (PoseArray) — DLO endpoints in world frame for manipulation
- `/dlo_manipulation/enable` (SetBool service) — Enable/disable autonomous following

## Code Conventions

- C++ standard: C++17 with `-O3` optimization for perception
- Commit messages: conventional commits (`feat:`, `fix:`, `docs:`, etc.), both Japanese and English
- Python linting: flake8 with max-line-length=150
- ROS2 nodes: C++ nodes inherit `rclcpp::Node`, Python from `rclpy.node.Node`
- All nodes declare parameters with `declare_parameter()`
- RGB-D synchronization uses `message_filters::ApproximateTimeSynchronizer`
- Perception nodes use `respawn=True, respawn_delay=3.0` in launch files
- Thread safety via `std::mutex` and `std::atomic` in C++ nodes
- Python entry points defined in `trackdlo_utils/setup.py` (9 console scripts)
- Sim vs real differences: downsample leaf size (0.008m vs 0.02m), max_iter (50 vs 20), dlo_pixel_width (10 vs 40)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HayatoShimada) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
