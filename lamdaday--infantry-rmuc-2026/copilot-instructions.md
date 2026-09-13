## infantry-rmuc-2026

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

Infantry robot firmware for RMUC 2026, targeting **STM32F407IG** (Cortex-M4, 168MHz). Uses FreeRTOS, STM32 HAL, and CMake with the STM32CubeCLT cross-compilation toolchain.

## Build Commands

```sh
# Configure (from repo root)
cmake -B build -DCMAKE_BUILD_TYPE=Debug

# Build
cmake --build build -j$(nproc)

# Clean
cmake --build build --target clean
```

The output binary is `build/Infantry_RMUC_2026.elf`. Flashing is done via VS Code's Cortex Debug launch configuration (`Cortex Debug (OpenOCD, CMSIS-DAP, STM32F407)`).

### Debug Setup

Install project-local OpenOCD once:
```sh
./tools/install_openocd.sh
```

Then use the VS Code launch configuration. If OpenOCD reports `CMSIS-DAP command CMD_INFO failed`, unplug and replug the probe. Live variable monitoring is available via `Cortex Live Watch` in the debug sidebar.

## Architecture

Three-layer design:

| Layer | Directory | Purpose |
|-------|-----------|---------|
| Application | `application/` | Robot behavior (4 apps, pub-sub only) |
| Module | `modules/` | Reusable drivers and algorithms |
| BSP | `bsp/` | STM32 HAL wrappers |

`Core/Src/` and `Drivers/` are STM32CubeMX-generated and should generally not be edited directly. Hardware configuration lives in `Infantry_RMUC_2026.ioc`.

### Application Layer

Four applications run as parallel FreeRTOS tasks at 1 kHz and **communicate exclusively through `modules/message_center/`** — no direct includes between apps:

- **`application/cmd/robot_cmd.c`** — Central hub. Receives RC/vision input, publishes control commands to the other three apps, subscribes to their feedback.
- **`application/chassis/chassis.c`** — Mecanum kinematics, power management, super-cap control.
- **`application/gimbal/gimbal.c`** — Pitch/yaw control with IMU gyro feedback.
- **`application/shoot/shoot.c`** — Friction wheels, loader, heat management.

### Key Configuration Files

- **`application/robot_board.h`** — Board selection macro (`GIMBAL_BOARD` / `CHASSIS_BOARD`). Only one may be defined at a time. Current default: `GIMBAL_BOARD`.
- **`application/robot_params.h`** — All mechanical and control parameters (wheel geometry, gimbal offsets, encoder limits, IMU calibration offsets, PID gains).
- **`application/robot_types.h`** — Shared data structures for inter-app messages (`Chassis_Ctrl_Cmd_s`, `Gimbal_Ctrl_Cmd_s`, `Shoot_Ctrl_Cmd_s`, etc.).

### Dual-Board Setup

The robot can split across two STM32 boards sharing a CAN bus:
- **Gimbal board**: runs gimbal + shoot + robot_cmd
- **Chassis board**: runs chassis

Switch by editing `application/robot_board.h` and recompiling each binary separately. Avoid CAN ID conflicts between the two boards.

### Message Center (Pub-Sub)

`modules/message_center/` provides up to 12 named topics. Apps call `MSG_DEV_MANAGER_REGIST_TOPIC` to publish and `MSG_DEV_MANAGER_REGIST_SUBCRIBER` to subscribe. Never bypass this with direct includes between apps.

### Motor Support

Multiple CAN-based motor protocols are supported under `modules/motor/`:
- **DJI** (M2006, M3508) — `DJImotor/`
- **DM** (DM4310) — `DMmotor/`
- **LK / HT / DR / YS** — separate directories

Motors are initialized with their CAN ID and feedback/control mode in the owning app's `Init()` function. CAN runs at 1 Mbps; monitor bus load when adding motors.

### IMU / INS

`modules/imu/` runs an EKF sensor-fusion task at 1 kHz using BMI088 (6-axis) + IST8310 (magnetometer). The fused attitude (quaternion + Euler angles) is published for gimbal and chassis. Calibration offsets are in `robot_params.h`.

The pre-built `modules/algorithm/AHRS.lib` is linked with `-Wl,--no-wchar-size-warning` because it was compiled with 2-byte `wchar_t`.

### Watchdog / Daemon

Every FreeRTOS task registers with `modules/daemon/`. If a task misses its deadline (~1.2 ms window), the daemon logs the overrun. Check daemon logs first when debugging timing issues.

### Adding a New Module or Feature

1. Add source files under `modules/<name>/` or `application/<name>/` — CMakeLists.txt globs them automatically.
2. If the module communicates across apps, define its topic in `message_center` and use pub-sub.
3. Register a daemon watchdog if the module runs as a FreeRTOS task.
4. Add parameters to `robot_params.h`; avoid magic numbers in source files.

---
> Source: [LamdaDay/Infantry_RMUC_2026](https://github.com/LamdaDay/Infantry_RMUC_2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
