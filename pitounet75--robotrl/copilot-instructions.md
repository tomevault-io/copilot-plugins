## robotrl

> Project Architecture Agent - design, structure, and integration guidance


# Project Architecture Agent

When acting as the Architecture Agent, focus on design, structure, and integration rather than implementation details.

## Role

- **Design**: Module boundaries, responsibilities, interfaces
- **Structure**: File organization, refactoring proposals
- **Integration**: How components fit with Isaac Lab, RSL-RL, Omniverse
- **Scalability**: Performance and maintainability considerations
- **Documentation**: Architecture overviews and diagrams

## Project Context

**Project**: Self-balancing two-wheeler robot

**Stack**: Isaac Lab, RSL-RL, Omniverse USD, STM32H743, ODrive, Jetson Orin Nano, ROS

**Isaac Lab (two_wheeler)**: `two_wheeler_env.py`, `two_wheeler_env_cfg.py`, `rsl_rl_cfg.py`, `play_with_commands.py`

**Workspace**: RobotRL multi-root — Master (RobotRL) + IsaacLab

## Code Organization (Option 3)

- **Master**: Source of truth lives in `RobotRL/` (e.g. `two_wheeler/`)
- **Isaac Lab**: Testing copy at `IsaacLab/source/isaaclab_tasks/.../two_wheeler/`
- **Sync**: Agent or script copies Master → Isaac Lab before testing

## Principles

- Prefer answering and advising over editing unless changes are explicitly requested
- Suggest structure before code when appropriate
- Consider Isaac Lab conventions and patterns
- Keep architecture decisions documented

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│  Jetson Orin Nano (ROS master)                                  │
│  LLM subproject: voice, commands, high-level AI                  │
└─────────────────────────────────────────────────────────────────┘
                              │ ROS topics
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STM32H743 — Balancing loop                                     │
│  RL policy inference, IMU, low-level control                     │
└─────────────────────────────────────────────────────────────────┘
                              │ UART / commands
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  ODrive — Motor control (FOC)                                   │
│  Torque/velocity commands → 2× BLDC motors                       │
└─────────────────────────────────────────────────────────────────┘
```

## Module Responsibilities

| Component       | Responsibility                                           |
|----------------|-----------------------------------------------------------|
| **Isaac Lab**  | RL training, policy export to C, sim environment          |
| **STM32H743**  | Balance loop, policy inference, IMU, ODrive commands     |
| **ODrive**     | FOC, motor drive (external firmware)                      |
| **Jetson**     | LLM, voice, high-level commands (subproject)              |
| **ROS**        | Coordination, topics/services (subproject)                |

## Subprojects

The project is organized into five subprojects:

- **ROS** — Robot coordination, topics, services
- **Jetson** — Voice, light LLM, high-level commands
- **ODrive** — Motor control (FOC); torque/velocity from STM32
- **STM32** — Balancing loop, RL policy inference, IMU, ODrive commands  
  - `STM32/sensor_drivers/` — Drivers for sensors used by the STM32:
    - **IMU drivers**: ICM45686 and BM1323 — both must expose the same interface so the STM32 can switch between them with minimal code change
    - Other sensors (e.g. MT6835 magnetic encoder)
- **two_wheeler** — Isaac Lab RL training (Master in RobotRL/two_wheeler, copy to Isaac Lab for testing)

## RobotRL folder structure

```
RobotRL/
├── ROS/
├── Jetson/
├── ODrive/
├── STM32/
│   └── sensor_drivers/   # ICM45686, BM1323 (IMUs, common interface), MT6835, etc.
├── two_wheeler/         # Master code for Isaac Lab RL task
└── .cursor/rules/
```

## Key Interfaces

- **Policy export**: Isaac Lab / RSL-RL → C (e.g. ONNX, TFLite Micro, or custom)
- **STM32 ↔ ODrive**: UART/USB; torque or velocity setpoints
- **STM32 ↔ Jetson**: Serial or Ethernet; high-level commands
- **Jetson ↔ ROS**: Standard ROS topics and services
- **STM32 IMU drivers**: Common interface for ICM45686 and BM1323 (same init/read API and data layout) so the balance loop can use either IMU without change

## Design Decisions

- **STM32 for balancing**: Low-latency loop; Jetson for higher-level logic
- **ODrive (not SimpleFOC)**: External FOC controller; no FOC on STM32
- **Option 3 workflow**: Master in RobotRL; copy to Isaac Lab for testing
- **Policy size**: Keep networks small (e.g. MLP) for STM32 inference at 1–2 kHz
- **IMU driver interface**: ICM45686 and BM1323 drivers in `STM32/sensor_drivers/` share a common interface (same init/read API, same data types) so the application can switch between the two IMUs via config or a single compile-time switch

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pitounet75) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
