## mechatronics-project

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Arduino Mega 2560-based maze-solving robot for UCF Mechatronics course. Solves a dual-maze problem: line maze (IR sensors + PID) then wall maze (ultrasonic sensors + PID), with automatic transition detection. The entire codebase is a single monolithic Arduino sketch at `MazeRobot/MazeRobot.ino`.

## Build & Upload

- **IDE:** Arduino IDE (no Makefile, PlatformIO, or CLI build system)
- **Board:** Arduino Mega 2560
- **Library dependency:** NewPing (install via Arduino Library Manager)
- **Serial monitor:** 115200 baud for debug output
- **No automated tests or CI** — verification is done via serial debug output and physical testing

## Code Architecture

### Single-file layout (MazeRobot.ino, ~1500 lines, 13 numbered sections)

| Section | Purpose |
|---------|---------|
| 1 | Debug toggle — compile-time `DEBUG_PRINT`/`DEBUG_PRINTF` macros via `#ifdef DEBUG` |
| 2 | Pin assignments — all hardware wiring constants |
| 3 | **Calibration constants — all tunable parameters live here** |
| 4 | Type definitions — `PIDController`, `Motor`, `Timer` structs with associated functions |
| 5 | State machine — `RobotState` enum (8 states), `TurnDir` enum |
| 6 | Global instances — motor/sensor/PID objects and state variables |
| 7 | Drive helpers — `driveTank`, `driveStop`, `driveBrake`, `startTurn` |
| 8 | Sensor functions — IR reading, ultrasonic reading, line position calculation |
| 9 | Line following — weighted centroid PID control |
| 10 | Wall following — dynamic wall selection, junction/dead-end handling |
| 11 | State transitions — mode switch logic, exit detection |
| 12 | Error/battery/stuck detection — safety systems and watchdog |
| 13 | `setup()` and `loop()` — initialization and state dispatch |

### State machine

```
LINE_FOLLOWING → MODE_SWITCH → WALL_FOLLOWING
     ↓                               ↓
  BACKING_UP ←──────────────────── TURNING
Any state → ERROR_STATE or EXIT_FOUND (terminal)
```

### Key design patterns

- **Non-blocking:** No `delay()` during navigation; `loop()` target < 50ms using `millis()`-based `Timer` structs
- **Isolated PID controllers:** Separate instances for line and wall modes with two-layer anti-windup (integral clamping + conditional integration)
- **EMA filtering:** Ultrasonic distances smoothed with alpha=0.3
- **Sensor scheduling:** Front ultrasonic every cycle; left/right alternate per cycle
- **Debug throttling:** Serial output limited to 200ms intervals to prevent flooding

## Important Conventions

- **All tuning happens in Section 3** (calibration constants). Never scatter magic numbers elsewhere.
- **Motor direction encoding:** Positive speed = forward, negative = backward, zero = coast. Trim offsets are clamped to preserve sign.
- **AVR limitation:** `snprintf` on AVR does not support `%f` for floats. Use `dtostrf()` or cast to int for debug output.
- **Naming:** `camelCase` for functions, `UPPER_CASE` for constants, descriptive pin names in Section 2.
- **Memory:** Stack buffers limited to 120 bytes for `snprintf`. No dynamic allocation.

## Hardware Reference

- **Motors:** 4x DC gear motors, skid-steer (tank drive), driven by 2x L298N H-bridge modules
- **Sensors:** 4x analog IR reflectance (A0-A3) for line detection, 3x HC-SR04 ultrasonic (front/left/right) for wall detection
- **LEDs:** 6 status indicators (power, error, line mode, wall mode, right wall, left wall)
- **Battery monitor:** Voltage divider on A4
- Full pin mapping is in Section 2 and `README.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stephennmiller) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
