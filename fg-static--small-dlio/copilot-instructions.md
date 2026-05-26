## small-dlio

> This repository is a ROS 2 colcon workspace. The only current package is `dlio`:

# Repository Guidelines

## Project Structure & Module Organization

This repository is a ROS 2 colcon workspace. The only current package is `dlio`:

- `src/dlio/src/`: C++ node implementations, including `main.cpp` and `odom_node.cpp`.
- `src/dlio/include/small_dlio/`: public headers and shared point/state types.
- `src/dlio/config/config.yaml`: runtime parameters for frames, topics, extrinsics, GICP, and observer gains.
- `src/dlio/launch/full.launch.py`: primary launch file. RViz is optional; Foxglove can consume the published ROS topics directly.

Generated build artifacts live in `build/`, `install/`, and `log/`; do not edit them.

## Build, Test, and Development Commands

Source dependencies before building:

```bash
source /opt/ros/jazzy/setup.bash
source /home/goose/fastlio/livox_ws/install/setup.bash
```

Build the package:

```bash
colcon build --packages-select dlio --event-handlers console_direct+
```

Run the node:

```bash
source install/setup.bash
ros2 launch dlio full.launch.py rviz:=false
```

Replay the current Mid360 test bag:

```bash
ros2 bag play /home/goose/fastlio/rosbag2_mid360_10hz --clock
```

## Coding Style & Naming Conventions

Use C++17 and the existing brace/indentation style in `odom_node.cpp`: 4 spaces, no tabs, and concise comments only where they clarify non-obvious logic. Keep ROS node members suffixed with `_` (`state_`, `pub_odom_`, `odom_frame_`). Preserve existing namespace `small_dlio` even though the ROS package name is `dlio`.

## Testing Guidelines

There is no formal test suite yet. Minimum validation is:

```bash
colcon build --packages-select dlio
ros2 launch dlio full.launch.py rviz:=false
ros2 topic list -t
```

When testing with a bag, confirm `/odom`, `/pose`, `/path`, `/tf`, `/tf_static`, and `/deskewed` appear. Use Foxglove fixed frame `odom`; `/deskewed` is published in the odom frame.

## Commit & Pull Request Guidelines

Recent commits use short Conventional Commit-style prefixes such as `feat:` and `refactor:`. Keep messages imperative and scoped, for example `feat: publish deskewed point cloud`.

Pull requests should include: what changed, how it was tested, relevant bag/topic assumptions, and any frame or parameter changes. Link issues when available and include screenshots only for visualization changes.

## Configuration Notes

Frame and topic names should come from `config.yaml`, not hard-coded strings. Keep Livox `CustomMsg` support sourced from the Livox workspace before build and run.

---
> Source: [FG-static/small_dlio](https://github.com/FG-static/small_dlio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-26 -->
