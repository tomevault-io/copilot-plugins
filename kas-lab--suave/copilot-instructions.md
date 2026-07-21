## suave

> SUAVE (Self-adaptive Underwater Autonomous Vehicle Exemplar) is a ROS 2 Humble exemplar for a single AUV pipeline-inspection mission: search for a pipeline, follow it, and inspect it. The repository separates the managed subsystem (vehicle mission functionality) from managing subsystems (adaptation logic), so adaptation managers can be swapped through standard ROS 2 interfaces.

# Repository Guidelines

## Project Overview

SUAVE (Self-adaptive Underwater Autonomous Vehicle Exemplar) is a ROS 2 Humble exemplar for a single AUV pipeline-inspection mission: search for a pipeline, follow it, and inspect it. The repository separates the managed subsystem (vehicle mission functionality) from managing subsystems (adaptation logic), so adaptation managers can be swapped through standard ROS 2 interfaces.

Runtime stack: ROS 2 Humble, Gazebo Harmonic, ArduSub/ArduPilot SITL, MAVROS, BehaviorTree.CPP, MROS2/Metacontrol, and Docker/Kasm.

## Project Structure & Module Organization

Core managed-system Python nodes live in `suave/suave/`, with launch files and sim config in `suave/launch/` and `suave/config/`. Monitoring nodes are in `suave_monitor/`, missions and mission configs in `suave_missions/`, metrics in `suave_metrics/`, auxiliary tools in `suave_tools/`, and experiment orchestration plus statistical analysis in `suave_runner/`. Managing subsystems are under `suave_managing/`, including `suave_none`, `suave_random`, `suave_metacontrol`, and the C++ BehaviorTree.CPP package `suave_bt`. Custom services are defined in `suave_msgs/srv/`. Tests are usually in each package's `test/` directory. Docker assets are in `docker/`, runner scripts in `runner/`, and documentation in `docs/source/`.

## Build, Test, and Development Commands

Run anything related to SUAVE execution inside the `suave_runner` container, including tests, ROS launches, `colcon`, and direct `pytest` runs. The host machine is not assumed to have SUAVE or ROS dependencies installed. Use the container's default sourced workspace configuration; do not override `PYTHONPATH`, `ROS_LOG_DIR`, or similar ROS/Python environment variables unless the user explicitly asks.

Default container command pattern:

```bash
docker exec suave_runner bash -lc 'cd /home/ubuntu-user/suave_ws && source /opt/ros/humble/setup.bash && source install/setup.bash && <command>'
```

Run ROS build and test commands from the ROS workspace root inside the container, `/home/ubuntu-user/suave_ws`, not from inside `src/suave`.

```bash
source /opt/ros/humble/setup.bash
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install
colcon build --symlink-install --executor sequential --parallel-workers 1
colcon build --symlink-install --packages-select <package_name>
source install/setup.bash
colcon test --packages-select <package_name> --event-handlers console_direct+
colcon test-result --verbose
python3 -m pytest -q <package>/test
```

After changing package data, launch files, setup metadata, or installed config files, rebuild with `colcon build --symlink-install` so installed resources are refreshed.

## Running SUAVE

Run SUAVE from inside the `suave_runner` container using the default workspace environment.

Use `cd runner && ./example_run.sh` for a full example. For manual runs:

```bash
# ArduSub SITL in a separate terminal
sim_vehicle.py -L RATBeach -v ArduSub --model=JSON --console

# Simulation
ros2 launch suave simulation.launch.py x:=-17.0 y:=2.0

# Mission, defaulting to no adaptation manager
ros2 launch suave_missions mission.launch.py

# Mission with a specific manager
ros2 launch suave_missions mission.launch.py adaptation_manager:=bt result_filename:=measurement_1
```

Valid `adaptation_manager` values are `none`, `metacontrol`, `random`, and `bt`. The preferred campaign runner is `ros2 launch suave_runner suave_runner.launch.py`; its config is `suave_runner/config/runner_config.yml` and results default to `~/suave/results/`. The shell runner is `cd runner && ./runner.sh [true|false] [metacontrol|random|none|bt] [time|distance] <runs>`, with `headless_runner.sh` using `screen` instead of `xfce4-terminal`.

MAVROS default FCU URL is `udp://0.0.0.0:14550@14555`, avoiding the need for `sim_vehicle --out=...`.

## Docker

```bash
docker run -it --shm-size=512m -p 6901:6901 -e VNC_PW=password --security-opt seccomp=unconfined ghcr.io/kas-lab/suave:main
./build_docker_images.sh
docker build -t suave-headless:dev -f docker/dockerfile-suave-headless .
docker build --check -f docker/dockerfile-suave-headless .
docker build --check --build-arg BASE_IMAGE=kasm-jammy:dev -f docker/dockerfile-suave .
```

Dockerfiles are intentionally lowercase as `docker/dockerfile-*`. When checking `docker/dockerfile-suave` without a local base, GHCR may return `denied`; use `--build-arg BASE_IMAGE=kasm-jammy:dev` after building the base locally. GUI image results are under `/home/kasm-user/suave/results`; headless image results are under `/home/ubuntu-user/suave/results`. The headless image has no `CMD`, so pass the runner command explicitly on `docker run`.

Version pins live in `docker/versions.env`, and Python package versions live in repository-root `requirements.txt`. `build_docker_images.sh` sources `versions.env` and forwards SHAs as build args.

## ROS Interfaces for Managing Subsystems

A compliant managing subsystem must interact with `/diagnostics` (`diagnostic_msgs/DiagnosticArray`), `/task/request` and `/task/cancel` (`suave_msgs/srv/Task`), and system_modes `ChangeMode` services: `/f_maintain_motion/change_mode`, `/f_generate_search_path/change_mode`, and `/f_follow_pipeline/change_mode`.

When adding a managing subsystem, include SUAVE's base launch with `task_bridge` disabled and wire the new package into `suave_missions/launch/mission.launch.py` behind an `adaptation_manager` condition.

## Coding Style & Naming Conventions

Python packages use `ament_python`, `rclpy`, four-space indentation, snake_case modules/functions/parameters, and PascalCase classes. Python code must pass flake8 and pep257. Add this Apache-2.0 header to Python files:

```python
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

Nodes subclass `rclpy.node.Node`, declare/read ROS parameters in `__init__`, expose module-level `main()` entry points, and register console scripts in `setup.py`. Use `extras_require={'test': ['pytest']}` rather than deprecated `tests_require`. Keep imports PEP8/flake8-clean and add pep257 docstrings for new public modules, classes, and functions. Avoid type hints unless they add clear value to the surrounding code.

C++ code in `suave_bt` targets C++17 with `-Wall -Wextra -Wpedantic`; package headers live under `include/suave_bt/`, implementations under `src/suave_bt/`, BT node names are registered in `src/suave_bt.cpp`, and XML trees live in `bts/`. `suave_msgs` is an `ament_cmake` package.

Launch files are Python ROS launch descriptions installed via `glob` in `setup.py` or CMake. Keep MAVROS launch args such as `fcu_url`, `gcs_url`, `mavros_config_yaml`, and `mavros_pluginlists_yaml` overrideable. New substantive files should follow nearby Apache-2.0 header style where present.

## Testing Guidelines

Python tests use `pytest` plus ROS linters such as `ament_flake8`, `ament_pep257`, and `ament_copyright`. Name tests `test_*.py` and place them in the affected package's `test/` directory. For Python changes, run `colcon test --packages-select <pkg> --event-handlers console_direct+` when ROS dependencies are available. For CMake, C++, or message changes, build the affected package before testing. Note any skipped tests when Gazebo, ArduSub, MAVROS, Docker, or other simulator dependencies are unavailable.

Use targeted checks when applicable:

```bash
python3 -m py_compile <launch_file.py>
bash -n <script>
docker build --check -f <dockerfile> .
```

If `setup.py` `data_files` or console script entry points change, verify they are installed correctly after build.

## Documentation

`docs/source/` is a Sphinx GitHub Pages site that mirrors README content. Keep both in sync when changing installation steps, Docker commands, runner usage, architecture, extension guidance, implementations, metrics, troubleshooting, related work, citing, or API docs.

The VCS dependencies file is `suave.repos`. The old name `suave.rosinstall` is obsolete and should not be referenced.

## Commit & Pull Request Guidelines

Recent commits use short imperative subjects, for example `add statistical_analysis` or `pass mission_config to task_bridge_none`; an emoji prefix appears occasionally for fixes. Keep commits focused and mention the affected package when useful. Pull requests should describe behavior changes, list test commands and results, link related issues, and include screenshots or logs for simulator, UI, Docker, or mission-run changes.

## Agent-Specific Instructions

Before editing, check `git status --short` and preserve unrelated user changes. Prefer `rg` for searches and keep changes scoped to the relevant ROS package. For SUAVE validation, always run tests and runtime checks inside `suave_runner` with the container's default sourced environment. Before finishing, run `git status --short` again and distinguish your edits from pre-existing uncommitted experiment, simulator, or Docker changes. Do not revert unrelated work.

---
> Source: [kas-lab/suave](https://github.com/kas-lab/suave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
