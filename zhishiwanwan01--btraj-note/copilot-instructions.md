## btraj-note

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
This is `btrajSim_2025-08`, a ROS-based UAV trajectory planning simulation framework called Btraj. It generates safe, dynamically feasible trajectories in unknown environments using a two-stage approach: front-end path finding (Fast Marching* or A*) and back-end trajectory optimization with Bezier curves.

The system implements the algorithm from "Online Safe Trajectory Generation For Quadrotors Using Fast Marching Method and Bernstein Basis Polynomial" (Fei Gao et al., ICRA 2018).

## Build System & Dependencies
- **ROS Framework**: ROS Melodic (Ubuntu 18.04) - updated from Kinetic
- **Build System**: CMake with `catkin_make`
- **Key Dependencies**:
  - libarmadillo-dev (linear algebra library)
  - PCL (Point Cloud Library)
  - Eigen3
  - Mosek (QP solver - requires academic license)
  - fast_methods (Fast Marching implementation)
  - sdf_tools (Euclidean distance field computation)

## Development Commands

### Docker Environment (Recommended)
```bash
# From repository root - Build and start ROS Melodic container
./build.sh  # Auto-detects Windows IP for X11 forwarding via SSH

# Enter running container for development
./start.sh

# Inside container - build workspace
cd /root/catkin_ws
catkin_make
source devel/setup.bash
```

**Important**: `build.sh` automatically detects the Windows client IP address when connecting via SSH to configure X11 display forwarding. The `DISPLAY_IP` environment variable is set to `<detected-windows-ip>:0.0` and passed to docker-compose.

### Native Build (Without Docker)
```bash
# Install dependencies
sudo apt-get install libarmadillo-dev ros-melodic-desktop-full

# Build from catkin workspace root (NOT repository root)
cd catkin_ws
catkin_make
source devel/setup.bash
```

### Running Simulation
```bash
# Launch main simulation (includes rviz visualization)
roslaunch bezier_planer simulation.launch

# Use 3D Nav Goal tool in rviz to set waypoints:
# - Click tool, press left mouse button on position
# - Drag up/down while holding left button for height
# - Release to send target
```

### Configuration
- Set path finding algorithm in launch file: `is_use_fm` (true=Fast Marching*, false=A*)
- Map parameters, velocity/acceleration limits configurable in `simulation.launch`

## Architecture

### System Data Flow
The planning pipeline follows this sequence:
1. **User Input** → RViz 3D Nav Goal → `waypoint_generator` → publishes waypoints
2. **Sensing** → `random_forest_sensing` → publishes point cloud map based on sensor range
3. **Odometry** → `odom_generator` → simulates robot state feedback
4. **Planning** → `b_traj_node` receives waypoints + map + odometry:
   - Front-end: Generates collision-free path (FM* or A*)
   - Back-end: Optimizes Bezier trajectory within flight corridors
   - Publishes `PolynomialTrajectory` (Bezier coefficients)
5. **Execution** → `b_traj_server` → converts coefficients to real-time position commands
6. **Control Loop** → position commands → `odom_generator` → closes simulation loop

### Core Classes (catkin_ws/src/Btraj/include/)
- **TrajectoryGenerator** (`trajectory_generator.h`): QP solver interface using Mosek. Main function `BezierPloyCoeffGeneration()` takes flight corridors, constraints (vel/acc limits), and MQM matrices to generate optimal Bezier coefficients.
- **Bernstein** (`bezier_base.h`): Pre-computes basis transformation matrices (Bernstein↔Monomial), cost matrices (MQM), and combinatorial constants. Must be initialized before trajectory generation.
- **gridPathFinder** (`a_star.h`): A* implementation on 3D grid using multimap priority queue. Uses Manhattan/Euclidean/Diagonal heuristics.

### ROS Node Architecture (from simulation.launch)
The `bezier_planer` package compiles to 4 executables (see CMakeLists.txt):
- **b_traj_node** (b_traj_node.cpp): Main planner integrating path finding + trajectory optimization
  - Subscribes: `/waypoint_generator/waypoints`, `/odom/fake_odom`, `/random_forest_sensing/random_forest`
  - Publishes: `/position_cmd` (PositionCommand), `/b_traj_node/trajectory` (PolynomialTrajectory)
- **b_traj_server** (traj_server.cpp): Trajectory execution - converts polynomial coefficients to position commands
- **odom_generator** (odom_generator.cpp): Simulates odometry from position commands
- **random_forest_sensing** (random_forest_sensing.cpp): Generates random obstacle environment + sensor simulation

**Supporting packages** (catkin_ws/src/plan_utils/):
- `waypoint_generator`: Converts RViz goals to waypoint messages
- `odom_visualization`: Displays robot state in RViz
- `rviz_plugins`: Custom 3D Nav Goal tool (drag for height)
- `quadrotor_msgs`: Message definitions (PolynomialTrajectory, PositionCommand)

## Key Implementation Details

### b_traj_node.cpp Callback Structure
Three main callbacks drive the planning system:
- `rcvOdometryCallbck()`: Updates `_start_pt`, `_start_vel`, `_start_acc` from odometry
- `rcvPointCloudCallBack()`: Inflates obstacles into collision map, triggers replanning if `checkExecTraj()` detects collision
- `rcvWaypointsCallback()`: Receives goal, calls `trajPlanning()` to generate trajectory

### Trajectory Optimization Parameters (simulation.launch)
- `poly_order` (8): Bezier polynomial order - higher = smoother but more control points
- `min_order` (2.5): Optimization target - fractional values interpolate between derivatives (2.5 = blend acceleration + jerk minimization)
- `is_use_fm` (true/false): Algorithm selection - Fast Marching* for smooth paths, A* for faster computation

### Mosek Integration
- Academic license required from mosek.com
- Place `mosek.lic` in `~/mosek/` directory (host) or mount to `/root/mosek/` (container)
- Docker volume mount: `../mosek:/root/mosek` in docker-compose.yml
- Linked via CMakeLists.txt: `link_directories(third_party/mosek/lib/mosek8_1)` and `mosek64` library

### Third-Party Libraries (catkin_ws/src/Btraj/third_party/)
- **fast_methods**: Fast Marching* implementation (FM* grid-based path finding)
- **sdf_tools**: Euclidean signed distance field computation for collision detection
- **mosek**: Commercial QP solver (requires license)

### X11 Display Issues
```bash
# If RViz fails to start, check X11 forwarding:
echo $DISPLAY
xhost +local:docker  # On host machine
```

### Container Development
```bash
# Enter running container:
docker exec -it btraj-sim-melodic bash

# Rebuild after code changes:
cd /root/catkin_ws && catkin_make
source devel/setup.bash
```

### Parameter Tuning
- **Algorithm Selection**: `planning/is_use_fm` in launch file (Fast Marching* vs A*)
- **Map Bounds**: `map/x_size`, `map/y_size`, `map/z_size` (default: 50x50x5m)
- **Trajectory Limits**: `planning/max_vel`, `planning/max_acc` (default: 2.0 m/s, 2.0 m/s²)
- **Polynomial Order**: `optimization/poly_order` (default: 8th order Bezier curves)
- **Safety Margin**: `map/margin` (default: 0.2m obstacle inflation)

## Citation
If using this for academic research, cite the original paper:
```bibtex
@inproceedings{Fei2018ICRA,
    Author = {F. Gao and W.Wu and Y. Lin and S. Shen},
    Title = {Online Safe Trajectory Generation For Quadrotors Using Fast Marching Method and Bernstein Basis Polynomial},
    Booktitle = {Proc. of the {IEEE} Intl. Conf. on Robot. and Autom.},
    Address = {Brisbane, Australia},
    Month = May,
    Year = {2018}
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zhishiwanwan01) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
