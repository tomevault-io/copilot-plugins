## ax-mycobotpi

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository contains control software for the **MyCobot 280 PI** robotic arm with visual servoing capabilities. The system uses a Raspberry Pi for robot control and can operate in two modes:
1. **Pi-only mode**: Vision and control both run on the Raspberry Pi
2. **Mac+Pi mode**: Vision processing runs on Mac, control commands sent to Pi via SSH

## Hardware Setup

- **Robot**: MyCobot 280 PI (6-DOF robotic arm)
- **Connection**: Serial port `/dev/ttyAMA0` at 1000000 baud
- **Camera**: Logitech C920 webcam for visual servoing
- **Network**: Pi accessible at `192.168.2.2` (user: `er`)

## Key Commands

### Robot Control (on Raspberry Pi)

```bash
# Move robot to zero position
python3 zero_robot.py

# Move robot to predefined resistor position (sequential movements)
python3 move_to_resistor.py

# Control robot with specific joint angles via command line
python3 robot_controller_pi.py J1 J2 J3 J4 J5 J6
# Example: python3 robot_controller_pi.py 0 -30 -45 0 0 0
```

### Visual Servoing

```bash
# Run visual servo control loop (Pi only)
python3 visual_servo_resistor.py

# Run visual servo from Mac (requires SSH to Pi)
python3 visual_servo_mac.py
```

### Camera Operations

```bash
# Capture image from webcam
python3 capture_camera.py
# Saves to: /Users/jacobmccarran_ax/Downloads/robot/camera_capture.jpg (Mac)
# or /home/er/camera_capture.jpg (Pi)
```

## Architecture

### Core Components

1. **robot_controller_pi.py** - Low-level robot interface
   - Accepts 6 joint angles as command-line arguments
   - Connects to MyCobot via serial
   - Reports current and final positions
   - Used by both manual control and visual servoing systems

2. **visual_servo_resistor.py** - Pi-based visual servoing
   - Full control loop running on Raspberry Pi
   - Image capture → Analysis → Movement calculation → Robot control
   - Uses OpenCV for vision (HSV color detection, contour analysis)
   - Iterative approach with movement limiting (max 50mm position, 30° orientation per step)

3. **visual_servo_mac.py** - Distributed visual servoing
   - Vision processing on Mac (faster, development-friendly)
   - Robot control on Pi via SSH
   - Simplified joint-space control (max 15° per joint per iteration)
   - Saves visualization images with detected robot/target positions

### Visual Servoing Strategy

Both visual servo implementations follow this control loop:
1. **Capture** image from camera
2. **Detect** robot arm end effector (blue/light blue HSV color detection)
3. **Detect** target resistor (edge detection + contour analysis for rectangular components)
4. **Calculate** error between robot position and target in pixel space
5. **Map** pixel error to joint angle changes (simplified inverse kinematics)
6. **Limit** movement to prevent large jumps (safety)
7. **Execute** movement and repeat until close enough (threshold < 15-50 pixels)

### Connection Parameters

All robot connections use:
```python
mc = MyCobot('/dev/ttyAMA0', 1000000)
```

The pymycobot library must be available at `/home/er/pymycobot/` on the Pi.

## Joint Angles and Coordinate System

- **6 joints**: Base (J1), Shoulder (J2), Elbow (J3), Wrist Roll (J4), Wrist Pitch (J5), Wrist Yaw (J6)
- **Range**: Most joints -170° to +170°, J5 is -180° to +180°
- **Speed**: Range 1-100, recommended 20-40 for safe operation
- **Zero position**: All joints at 0° (straight up configuration)

## Important Implementation Notes

### Vision Processing
- Robot arm detection uses HSV range: [80,50,50] to [130,255,255] (blue tones)
- Resistor detection looks for rectangular contours with aspect ratio 1.5-8
- Fallback target: center of circuit board area if no resistor detected
- Mac version saves visualization to `analysis_visualization.jpg`

### Movement Safety
- Always limit joint angle changes per iteration to prevent unsafe movements
- visual_servo_resistor.py: 50mm max position change, 30° max orientation change
- visual_servo_mac.py: 15° max joint angle change
- Default movement speed: 30 (safe middle range)

### SSH Communication (Mac mode)
```bash
ssh er@192.168.2.2 "cd ~ && python robot_controller_pi.py J1 J2 J3 J4 J5 J6"
```

## File Purposes

- **zero_robot.py**: Reset to zero position
- **move_to_resistor.py**: Predefined 4-step sequence to resistor position
- **try_2_resistor.py**: Alternative resistor positioning script
- **robot_controller_pi.py**: Command-line robot control interface
- **capture_camera.py**: Standalone camera capture utility
- **visual_servo_resistor.py**: Complete Pi-based visual servoing
- **visual_servo_mac.py**: Mac-based vision with Pi control
- **camera_capture.jpg**: Current camera frame (updated by capture scripts)
- **analysis_visualization.jpg**: Annotated image showing detected robot/target (Mac mode)

---
> Source: [jacobmcc-ax/Ax-myCobotPi](https://github.com/jacobmcc-ax/Ax-myCobotPi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
