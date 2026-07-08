## car-project

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Raspberry Pi robot car controller with camera capabilities. Python project that drives a two-motor car via GPIO pins using an L298N motor driver (or similar H-bridge) and provides video streaming and image capture through the Raspberry Pi Camera Module. Runs exclusively on Raspberry Pi hardware due to `RPi.GPIO` dependency and camera requirements.

## Running

All entry points must be run on the Raspberry Pi with `RPi.GPIO` and `websockets` installed. Camera features additionally require `rpicam-still` and `rpicam-vid` (part of the standard Raspberry Pi OS camera stack).

```bash
# Terminal-based keyboard control (arrow keys / WASD)
python keyboard_control.py

# WebSocket server for remote control (listens on 0.0.0.0:8765)
# Supports motor commands + image capture via "capture" command
python websocket_server.py

# MJPEG HTTP streaming server (listens on 0.0.0.0:8080)
# View at http://<raspberry-pi-ip>:8080/
python camera_server.py
```

There is no build step, no test suite, no linter configuration, and no `requirements.txt`. Core dependencies are `RPi.GPIO` and `websockets`. Camera functionality requires Raspberry Pi OS camera utilities (`rpicam-still`, `rpicam-vid`).

## Architecture

```
Control Layer          →   Core Layer         →   Hardware
────────────────────────────────────────────────────────────
keyboard_control.py         │                       │
websocket_server.py    → MotorController      → RPi.GPIO
camera_server.py        → camera.py           → Pi Camera
                            │                       │
                        config.py             GPIO pins
```

### Motor Control

- **`config.py`** — GPIO pin assignments (BCM numbering) and defaults (speed 80%, PWM 100Hz, joystick dead zone 0.1). Motor A: ENA=27, IN1=23, IN2=24. Motor B: ENB=16, IN3=26, IN4=6.
- **`motor_controller.py`** — `MotorController` class. Wraps all GPIO setup, PWM, movement commands (forward/backward/left/right/stop), speed control (0–100%), and low-level `drive()` method for differential motor control. All control interfaces should use this class.
- **`keyboard_control.py`** — Reads raw terminal input for arrow keys and WASD. Standalone entry point.
- **`websocket_server.py`** — Async WebSocket server (port 8765). Accepts JSON commands for motor control and camera capture. Commands: `forward`, `backward`, `left`, `right`, `stop`, `speed`, `joystick`, `capture`. Returns JSON responses with status. Designed for Android app clients.

### Camera System

- **`camera.py`** — Image capture module. Exports `capture_image(width, height, quality)` function that uses `rpicam-still` to capture a single frame and return base64-encoded JPEG. Default: 640x480 at quality 80. Returns dict with `success` bool and either `data` (base64 string) or `error` message.
- **`camera_server.py`** — Standalone MJPEG HTTP streaming server (port 8080). Uses `rpicam-vid` to generate real-time MJPEG stream at 640x480@15fps. Serves two endpoints: `/` (HTML viewer) and `/stream` (raw MJPEG multipart stream). Run independently from motor control—camera and motors can operate simultaneously.

### WebSocket Camera Commands

The `websocket_server.py` integrates the `camera` module to support image capture over WebSocket:

**Request:**
```json
{
  "command": "capture",
  "width": 640,
  "height": 480,
  "quality": 80
}
```

All parameters are optional (defaults: 640x480, quality 80).

**Response (success):**
```json
{
  "status": "ok",
  "command": "capture",
  "image": "<base64-encoded-jpeg>",
  "width": 640,
  "height": 480
}
```

**Response (failure):**
```json
{
  "status": "error",
  "command": "capture",
  "message": "capture failed"
}
```

### MJPEG HTTP Streaming

The `camera_server.py` provides real-time video streaming over HTTP, designed for Android app integration alongside WebSocket motor control.

**Server Configuration:**
- Port: 8080
- Resolution: 640x480
- Framerate: 15 fps
- Codec: MJPEG

**Endpoints:**

| Endpoint | Content-Type | Description |
|----------|--------------|-------------|
| `/` | `text/html` | HTML page with embedded video viewer |
| `/stream` | `multipart/x-mixed-replace` | Raw MJPEG stream for direct consumption |

**Android Integration:**

Run both servers simultaneously for full control + video:
```bash
# Terminal 1 - Motor control
python websocket_server.py

# Terminal 2 - Video stream
python camera_server.py
```

In your Android app, use the same Pi IP for both connections:
```kotlin
val piIp = "192.168.1.100"

// WebSocket for motor control
val wsUrl = "ws://$piIp:8765"

// MJPEG stream URL - display in WebView or MjpegView library
val streamUrl = "http://$piIp:8080/stream"
```

**Stream Lifecycle:**
- Stream starts automatically when a client connects to `/stream`
- Stream stops when the client disconnects (closes HTTP connection)
- Multiple clients can connect simultaneously (each spawns its own camera process)
- Server handles graceful shutdown on SIGINT/SIGTERM (Ctrl+C)

## Key Conventions

- GPIO uses BCM pin numbering (`GPIO.setmode(GPIO.BCM)`).
- Turning is differential: left/right spin motors in opposite directions (tank-style steering).
- All entry points call `motor.cleanup()` on exit to release GPIO resources. New entry points must do the same.
- The WebSocket server uses a module-level global `motor` instance shared across client connections.
- Camera commands use `rpicam-still` for stills and `rpicam-vid` for streaming (libcamera-based tools, standard on Raspberry Pi OS).
- The MJPEG server runs independently on a separate port (8080) and can stream video while the WebSocket server handles motor control (8765).

## Planned Extensions

Sensor integration (`sensors.py`) and autonomous driving mode (`autonomous.py`).

---
> Source: [gonalc/car-project](https://github.com/gonalc/car-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
