## return-to-sender-arduino

> A web-based interface for controlling a 4-axis CNC Shield v3 stepper motor setup using an Arduino Uno R4. The project enables real-time motor control, choreography recording/playback, and parameter tuning via a browser.

# Project Context: CNC Shield Controller

## Overview
A web-based interface for controlling a 4-axis CNC Shield v3 stepper motor setup using an Arduino Uno R4. The project enables real-time motor control, choreography recording/playback, and parameter tuning via a browser.

## Tech Stack
- **Frontend:** HTML5, CSS3, JavaScript (ES6 Modules)
    - `app.js`: Main controller.
    - `comms.js`: WebSocket communication.
    - `state.js`: Global state management.
    - `ui.js`: DOM updates (Timeline, Lists).
    - `choreography.js`: Playback loop and logic.
    - `storage.js`: LocalStorage & IndexedDB.
    - `kinematics.js`: Geometry math.
- **Backend:** Node.js, Express, `ws` (WebSocket), `serialport`
- **Firmware:** Arduino (C++), `AccelStepper` library, optimized for Arduino Uno R4
- **Communication:** Serial (115200 baud) for Arduino <-> Node.js, WebSocket for Browser <-> Node.js

## Hardware Configuration
- **Controller:** Arduino Uno R4 (Minima/WiFi)
- **Shield:** CNC Shield v3
- **Drivers:** 4x Stepper Drivers (A4988/DRV8825)
- **Motors:** 4x NEMA 17 Stepper Motors (Axes: X, Y, Z, A)
- **Spool:** 24mm Diameter
- **Microstepping:** 1/8 (configured via jumpers and software)

## Feature Status

### Core Motion Control
- [x] Independent control of 4 stepper motors (X, Y, Z, A)
- [x] Absolute (`M`) and Relative (`R`) positioning
- [x] Dynamic Speed and Acceleration configuration (x1000 scale in UI)
- [x] **Port Selection:** Select Arduino serial port from UI.
- [x] **Set Floor:** Sets current position as Zero (Hardware & Software).
- [x] **Set Ceiling:** Sets Max Height limit based on current position.
- [x] **STOP & Reset:** Immediate deceleration halt (`Q` command) and return to 0.
- [x] **Motor Direction Inversion:** Hardware-synced inversion toggle (sends `V` command).
- [x] **Motor Enable/Disable:** Auto-homes on disable.
- [x] **Config Sync:** Motor dimensions and limits persist on server and sync across devices.

### Advanced Features
- [x] **Choreography:** Record & Playback with Speed/Accel per keyframe.
- [x] **Audio Sync:** Load audio tracks, sync playback, scrub timeline.
- [x] **Loop & Rest:** Playback loops with optional Rest/Cool-down buffer (server-side logic).
- [x] **Project Management:** Quick Save/Load slots in LocalStorage.
- [x] **Visual Editor:** Drag-and-drop keyframes, virtual box pose editing.
- [x] **Infinite Recording:** Playback continues past end for extending sequences.
- [x] **Configuration Export/Import:** Save and load full hardware configuration (Mappings, Limits, Drivers) to JSON.
- [x] **Calibration Wizard:** Random-access corner adjustment UI.

## Architecture Notes
- **Modular JS:** The frontend is now split into ES6 modules to manage complexity.
- **Server-Side Choreography:**
    - The Raspberry Pi (`server.js` + `server-choreography.js`) is the master conductor.
    - It maintains the playback loop, executes keyframes, and sends serial commands.
    - Clients are purely visual/control interfaces that sync via WebSocket.
    - Closing the browser does NOT stop playback.
- **State Management:** `state.js` acts as the single source of truth for the Frontend. `ServerChoreography` holds the truth for the Backend.
- **Firmware Protocol:**
    - `M x y z a`: Move Absolute
    - `R x y z a`: Move Relative
    - `Q`: Quick Stop (Decelerate to 0)
    - `E 0/1`: Enable/Disable
    - `H`: Set Home (0)
- **Persistence:**
    - **Choreography:** `localStorage`
    - **Audio:** `IndexedDB` (to handle large binary blobs)
- **API Endpoints:**
    - `GET /api/ports`: List available serial ports.
    - `POST /api/connect`: Connect to a specific port.
    - `POST /api/command`: Send raw command (fallback).

## Performance Tuning
- **Microstepping:** UI allows selecting driver type and jumper config to calculate steps/mm.
- **Speed Limits:** UI caps speed at 30k steps/sec to prevent stalling.
- **Interpolation:** Frontend uses a `requestAnimationFrame` loop with velocity ramping to smooth out visual updates between status polls.

## Maintenance Rules
- **Update Context:** Every time a significant feature is implemented, an architectural change is made, or a major bug is fixed, this file (`GEMINI.md`) MUST be updated to reflect the new technical state of the project. This ensures AI agents and developers always have an accurate source of truth.

## Raspberry Pi Deployment

### Auto-Start Service
The Node.js server runs as a systemd service on the Pi:

**Service file:** `/etc/systemd/system/return-to-sender.service`
```ini
[Unit]
Description=Return to Sender Node.js Server
After=network.target

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi/return_to_sender
ExecStart=/usr/bin/npm start
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

**Useful commands:**
```bash
# Check status
systemctl status return-to-sender.service

# View logs
journalctl -u return-to-sender.service -n 50

# Restart the service
sudo systemctl restart return-to-sender.service

# Stop/Start
sudo systemctl stop return-to-sender.service
sudo systemctl start return-to-sender.service
```

### Audio Playback
- Audio is played server-side using `mpv` (or `ffplay` as fallback)
- Uploaded audio files are stored in `~/return_to_sender/uploads/`
- Audio config persisted in `uploads/audio-config.json`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/prismspecs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/prismspecs)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
