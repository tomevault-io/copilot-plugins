## mur-system-controller

> This repository is an ESP-IDF firmware project for `pwm_controller`. The top-level [CMakeLists.txt](/home/nolan/ECE494-MUR/pwm_controller/CMakeLists.txt) registers component roots in `drivers/`, `tasks/`, and `sys_common/`. Application startup lives in [main/pwm_controller.c](/home/nolan/ECE494-MUR/pwm_controller/main/pwm_controller.c). Component code is grouped by domain: `drivers/adc/` for hardware drivers, `tasks/SENSOR/` and `tasks/COMMS/` for FreeRTOS tasks, and `sys_common/` for shared types, configuration, and system-state helpers. Public headers live under each component’s `include/` directory.

# Repository Guidelines

## Project Structure & Module Organization
This repository is an ESP-IDF firmware project for `pwm_controller`. The top-level [CMakeLists.txt](/home/nolan/ECE494-MUR/pwm_controller/CMakeLists.txt) registers component roots in `drivers/`, `tasks/`, and `sys_common/`. Application startup lives in [main/pwm_controller.c](/home/nolan/ECE494-MUR/pwm_controller/main/pwm_controller.c). Component code is grouped by domain: `drivers/adc/` for hardware drivers, `tasks/SENSOR/` and `tasks/COMMS/` for FreeRTOS tasks, and `sys_common/` for shared types, configuration, and system-state helpers. Public headers live under each component’s `include/` directory.

Use `system_monitor/` as the structural model for new code in this project. When porting behavior, treat `sensor_controller/` as the feature reference only.

## Build, Test, and Development Commands
Run all commands from the repository root with ESP-IDF exported.

- `idf.py build` compiles the full firmware and validates component wiring.
- `idf.py flash monitor` programs the target and opens the serial monitor.
- `idf.py fullclean build` clears cached artifacts before a clean rebuild.
- `idf.py menuconfig` updates target-specific configuration stored in `sdkconfig`.

The `build/` directory is generated output and should not be edited manually.

## Coding Style & Naming Conventions
Follow the existing C style: 4-space indentation, braces on the same line for functions and control blocks, and concise inline comments only where behavior is non-obvious. Use `snake_case` for variables, functions, and fields (`update_system_status`), `UPPER_SNAKE_CASE` for macros and constants (`LEAK_THRESHOLD_MV`), and keep task/component names aligned with their directories (`SENSOR`, `COMMS`). Place new public declarations in the component’s `include/` folder and keep component dependencies declared in its local `CMakeLists.txt`.

For implementation strategy, preserve `system_monitor`'s component boundaries and task flow. If `sensor_controller` has equivalent logic, port the behavior without copying its UART-centric structure. `pwm_controller` must communicate over SPI via the existing `tasks/COMMS/`, [spi.c](/home/nolan/ECE494-MUR/pwm_controller/tasks/COMMS/spi.c), and [packet.c](/home/nolan/ECE494-MUR/pwm_controller/tasks/COMMS/packet.c) path; do not add new UART transport code here.

## Testing Guidelines
No standalone automated test suite is committed yet; there is no `test/` directory or `ctest` target. At minimum, require a successful `idf.py build` before opening a PR. For hardware-facing changes, verify on target with `idf.py flash monitor` and describe the setup and observed behavior in the PR. When adding tests later, keep them near the affected component and name them after the module under test.

## Commit & Pull Request Guidelines
Recent history uses short, imperative summaries such as `added system status reporting to COMMS` and `fixed build issue`. Keep commit subjects brief, specific, and focused on one change. Pull requests should include a short problem statement, a summary of the fix, impacted components, and any hardware validation steps. Link the related issue when applicable and include serial logs or screenshots when the change affects runtime behavior.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nrmshaw100)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nrmshaw100)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
