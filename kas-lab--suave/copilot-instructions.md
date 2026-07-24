## suave

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SUAVE (Self-adaptive Underwater Autonomous Vehicle Exemplar) models a single AUV performing a pipeline inspection mission: searching for a pipeline, following it, and inspecting it. The repo cleanly separates a **managed subsystem** (vehicle mission functionality) from **managing subsystems** (adaptation logic), so different adaptation managers can be plugged in through standard ROS 2 interfaces.

Runtime stack: ROS 2 Humble · Gazebo Harmonic · ArduSub/ArduPilot SITL · MAVROS · BehaviorTree.CPP · MROS2/Metacontrol · Docker/Kasm.

## Repository Layout

| Path | Purpose |
|---|---|
| `suave/` | Managed subsystem: functionalities, launch files, sim config |
| `suave_monitor/` | Monitor nodes (thruster, battery, water visibility) — publish diagnostics |
| `suave_missions/` | Mission planners and launch files |
| `suave_metrics/` | Metrics collection |
| `suave_runner/` | Experiment runner and statistical analysis |
| `suave_tools/` | Auxiliary tools (PlotJuggler config, etc.) |
| `suave_msgs/` | Custom ROS service definitions (`Task.srv`, `GetPath.srv`) — `ament_cmake` |
| `suave_managing/suave_none/` | No-manager launch variant |
| `suave_managing/suave_random/` | Random managing subsystem |
| `suave_managing/suave_metacontrol/` | MROS2/Metacontrol managing subsystem |
| `suave_managing/suave_bt/` | BehaviorTree.CPP managing subsystem (C++17) |
| `docker/` | Dockerfile definitions and install scripts |
| `runner/` | Shell scripts for running experiments |

## Build & Test

Run anything related to SUAVE execution inside the `suave_runner` container, including tests, ROS launches, `colcon`, and direct `pytest` runs. The host machine is not assumed to have SUAVE or ROS dependencies installed. Use the container's default sourced workspace configuration; do not override `PYTHONPATH`, `ROS_LOG_DIR`, or similar ROS/Python environment variables unless the user explicitly asks.

Default container command pattern:

```bash
docker exec suave_runner bash -lc 'cd /home/ubuntu-user/suave_ws && source /opt/ros/humble/setup.bash && source install/setup.bash && <command>'
```

All build/test commands run from the **ROS workspace root inside the container** (`/home/ubuntu-user/suave_ws`), not from inside `src/suave`.

```bash
source /opt/ros/humble/setup.bash

# Install deps
rosdep install --from-paths src --ignore-src -r -y

# Build all
colcon build --symlink-install

# Low-memory build
colcon build --symlink-install --executor sequential --parallel-workers 1

# Build single package
colcon build --symlink-install --packages-select <package_name>

source install/setup.bash

# Test a package
colcon test --packages-select <package_name> --event-handlers console_direct+
colcon test-result --verbose

# Run Python tests directly (after sourcing)
python3 -m pytest -q <package>/test
```

Lint tests are pytest wrappers around `ament_flake8`, `ament_pep257`, and `ament_copyright`.

## Running SUAVE

Run SUAVE from inside the `suave_runner` container using the default workspace environment.

```bash
# Quick example
cd runner && ./example_run.sh

# ArduSub SITL (separate terminal)
sim_vehicle.py -L RATBeach -v ArduSub --model=JSON --console

# Simulation
ros2 launch suave simulation.launch.py x:=-17.0 y:=2.0

# Mission (default: no adaptation manager)
ros2 launch suave_missions mission.launch.py

# Mission with a specific manager
ros2 launch suave_missions mission.launch.py adaptation_manager:=bt result_filename:=measurement_1
# adaptation_manager values: none | metacontrol | random | bt

# Experiment runner (ROS2, config-file driven — preferred for campaigns)
ros2 launch suave_runner suave_runner.launch.py
# Config: suave_runner/config/runner_config.yml — controls experiments, disturbance timing, result_path

# Shell runner (simple positional args)
cd runner && ./runner.sh [true|false] [metacontrol|random|none|bt] [time|distance] <runs>
# headless_runner.sh is the same but uses screen instead of xfce4-terminal

# Results default to: ~/suave/results/
```

MAVROS default FCU URL: `udp://0.0.0.0:14550@14555` (avoids needing `sim_vehicle --out=...`).

## Docker

```bash
# Run GUI image
docker run -it --shm-size=512m -p 6901:6901 -e VNC_PW=password --security-opt seccomp=unconfined ghcr.io/kas-lab/suave:main

# Build all images locally
./build_docker_images.sh

# Build headless image (from repo root)
docker build -t suave-headless:dev -f docker/dockerfile-suave-headless .

# Syntax check
docker build --check -f docker/dockerfile-suave-headless .
docker build --check --build-arg BASE_IMAGE=kasm-jammy:dev -f docker/dockerfile-suave .
```

Dockerfiles are intentionally lowercase (`docker/dockerfile-*`). When checking `docker/dockerfile-suave` without a local base, GHCR may return `denied` — use `--build-arg BASE_IMAGE=kasm-jammy:dev` after building it locally.

**Image users and result paths:**
- GUI image (`suave:main`): user `kasm-user`, results at `/home/kasm-user/suave/results`
- Headless image (`suave-headless:main`): user `ubuntu-user`, results at `/home/ubuntu-user/suave/results`
- Headless image has no `CMD` — pass the runner command explicitly on `docker run`.

**Version pinning:** git SHAs live in `docker/versions.env`; Python package versions live in `requirements.txt` (repo root). `build_docker_images.sh` sources `versions.env` and forwards SHAs as `--build-arg`.

## ROS Interfaces for Managing Subsystems

A compliant managing subsystem must interact with:
- **`/diagnostics`** — `diagnostic_msgs/DiagnosticArray`
- **`/task/request`** and **`/task/cancel`** — `suave_msgs/srv/Task`
- **system_modes** `ChangeMode` services: `/f_maintain_motion/change_mode`, `/f_generate_search_path/change_mode`, `/f_follow_pipeline/change_mode`

When adding a new managing subsystem: include SUAVE's base launch with `task_bridge` disabled and wire the new package into `suave_missions/launch/mission.launch.py` via an `adaptation_manager` condition.

## Code Conventions

**Python (`ament_python` packages):**
- Python code must pass flake8 and pep257.
- Add the standard `Copyright 2026 KAS Lab` Apache-2.0 header to Python files:
  ```
  # Copyright 2026 KAS Lab
  #
  # Licensed under the Apache License, Version 2.0 (the "License");
  # you may not use this file except in compliance with the License.
  # You may obtain a copy of the License at
  #
  #     http://www.apache.org/licenses/LICENSE-2.0
  #
  # Unless required by applicable law or agreed to in writing, software
  # distributed under the License is distributed on an "AS IS" BASIS,
  # WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  # See the License for the specific language governing permissions and
  # limitations under the License.
  ```
- Nodes subclass `rclpy.node.Node`; declare/read ROS params in `__init__`; expose `main()` as console script entry point.
- `snake_case` for functions/methods/variables/file names; `PascalCase` for classes.
- No type hints unless they add clear value to surrounding context.
- `extras_require={'test': ['pytest']}` — do not use deprecated `tests_require`.
- PEP8/flake8-clean imports; pep257 docstrings on new public modules/classes/functions.

**C++ (`suave_bt`, `suave_msgs`):**
- C++17, `-Wall -Wextra -Wpedantic`. Headers under `include/suave_bt`, implementations under `src/suave_bt`.
- BT node names registered in `src/suave_bt.cpp`; XML trees in `bts/`.

**Launch / Config:**
- Launch files are Python ROS launch descriptions installed via glob in `setup.py` or CMake.
- Keep MAVROS launch args (`fcu_url`, `gcs_url`, `mavros_config_yaml`, `mavros_pluginlists_yaml`) overrideable.
- After changing mission configs or package data files, `colcon build --symlink-install` may be needed.

**Licensing:** Apache-2.0 headers are present in many files; new substantive files should follow the nearby package header style.

## Documentation

`docs/source/` is a Sphinx GitHub Pages site that mirrors README content — keep both in sync when updating installation steps, Docker commands, or runner docs. Pages: `installation`, `docker`, `run`, `architecture`, `extend`, `implementations`, `metrics`, `troubleshooting`, `related`, `citing`, `api`.

The VCS dependencies file is `suave.repos` (vcs format). The old name `suave.rosinstall` is obsolete — do not reference it.

## Before Finishing Changes

- `git status --short` — distinguish own edits from pre-existing uncommitted experiment/Docker changes; do not revert unrelated work.
- SUAVE validation must run inside `suave_runner` with the container's default sourced workspace environment; do not run ROS/SUAVE tests on the host.
- Python changes: `colcon test --packages-select <pkg> --event-handlers console_direct+`.
- C++/message changes: build then test.
- Launch file changes: `python3 -m py_compile <launch_file.py>`.
- Shell script changes: `bash -n <script>`.
- Dockerfile changes: `docker build --check`.
- If `setup.py` data_files or console script entry points change, verify they are installed correctly after build.
- Note any tests skipped because ROS/Gazebo/ArduSub/Docker dependencies are unavailable in the environment.

---
> Source: [kas-lab/suave](https://github.com/kas-lab/suave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
