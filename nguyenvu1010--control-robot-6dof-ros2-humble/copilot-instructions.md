## control-robot-6dof-ros2-humble

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ROS 2 Humble control system for a 6-DOF robotic arm with gripper. Supports both simulation (mock hardware) and real hardware via ESP32 over serial. The GUI and controller communicate through POSIX shared memory (not ROS topics) for low-latency IPC.

## Build & Run Commands

```bash
# Install system dependencies (first time only)
source setup_environment.sh

# Build all packages
colcon build

# Source the workspace
source install/setup.bash

# Launch (simulation mode, default)
ros2 launch my_arm_robot robot_bringup.launch.py

# Launch with real hardware
ros2 launch my_arm_robot robot_bringup.launch.py use_mock_hardware:=false serial_port:=/dev/ttyUSB0 baud_rate:=115200

# Build a single package
colcon build --packages-select <package_name>

# Build with dependencies
colcon build --packages-up-to <package_name>
```

ESP32 firmware is built separately with PlatformIO in `src/esp32_robot_pio/`.

## Architecture

### Package Map

```
src/
├── robot/                    # URDF/xacro, launch files, meshes, rviz config (CMake, pkg: my_arm_robot)
├── my_robot_hardware/        # ros2_control hardware interface plugin (C++, serial I/O to ESP32)
├── my_robot_controllers/     # CartesianVelocityController plugin (C++, FK/IK/trajectory orchestration)
├── simple_kinematics_lib/    # FK, IK, Jacobian library (C++, Eigen-based, namespace: srk)
├── simple_trajectory_lib/    # Cubic polynomial trajectory generator (C++, namespace: stl)
├── robot_gui/                # PyQt6 GUI application (Python, communicates via shared memory)
└── esp32_robot_pio/          # ESP32 firmware (PlatformIO/Arduino, stepper + encoder control)
```

### Data Flow

```
GUI (PyQt6, 30Hz) ←──shared memory──→ CartesianVelocityController (500Hz)
                                            ↕ hardware_interface
                                      RobotSystem (serial binary protocol)
                                            ↕ UART 115200
                                      ESP32 (steppers + AS5600 encoders)
```

### Shared Memory Layout (`/robot_control_shm`, ~322 bytes)

The controller writes feedback (joint positions/velocities, EE pose) and reads commands (control mode, targets). The GUI does the inverse. Defined in:
- C++: `src/my_robot_controllers/include/my_robot_controllers/shm/shm_msgs.hpp`
- Python: `src/robot_gui/config.py`

### Control Modes (set via shared memory)

| Mode | Value | Description |
|------|-------|-------------|
| IDLE | 0 | No motion |
| CARTESIAN_POSE | 1 | IK to reach target pose |
| TRAJECTORY | 2 | Execute trajectory with cubic interpolation |
| JOINT_MANUAL | 3 | Direct joint velocity commands |

### Serial Protocol (binary, defined in `protocol.hpp` / `protocol.h`)

- **Command (PC→ESP32):** header `0x55AA` + 6 float joint velocities + 1 float gripper + checksum
- **Feedback (ESP32→PC):** header `0xAA55` + 6 float positions + 6 float velocities + 1 float gripper + checksum

### Robot Structure

6 revolute joints (`Link_1_joint` through `Link_6_joint`) commanded via velocity, plus a gripper (`gripper_right_joint`) commanded via position. Joint limits: ±1.57 rad. Kinematic chain: `base_link` → `tool_center_point`.

### Plugin Registration

- Hardware: `my_robot_hardware_plugins.xml` exports `my_robot_hardware/RobotSystem`
- Controller: `my_robot_controllers_plugins.xml` exports `my_robot_controllers/CartesianVelocityController`

## Key Dependencies

- C++: Eigen3, libserial, ros2_control, hardware_interface, pluginlib
- Python: PyQt6, posix_ipc
- ROS 2: ros2-control, ros2-controllers, xacro, joint-state-broadcaster

## Conventions

- Controller config lives in `src/my_robot_controllers/config/controllers.yaml`
- Joint names must match exactly between URDF, controllers.yaml, and hardware interface
- Code comments are in Vietnamese

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NguyenVu1010) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
