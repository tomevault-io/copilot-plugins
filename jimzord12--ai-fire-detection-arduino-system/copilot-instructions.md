## ai-fire-detection-arduino-system

> This project is an **AI-powered fire detection system** built on the **Arduino UNO R4 WiFi**. It utilizes a sensor fusion approach, combining data from multiple DFRobot sensors (Smoke, Flame, VOC, CO, Temperature/Humidity) to detect fire hazards with high accuracy and low false alarm rates. The system uses **Edge Impulse** for training and deploying a TinyML neural network directly to the microcontroller.

# AI Fire Detection System - Project Context

## Project Overview
This project is an **AI-powered fire detection system** built on the **Arduino UNO R4 WiFi**. It utilizes a sensor fusion approach, combining data from multiple DFRobot sensors (Smoke, Flame, VOC, CO, Temperature/Humidity) to detect fire hazards with high accuracy and low false alarm rates. The system uses **Edge Impulse** for training and deploying a TinyML neural network directly to the microcontroller.

## Architecture
1.  **Hardware Layer**:
    *   **MCU**: Arduino UNO R4 WiFi (Renesas RA4M1 + ESP32-S3).
    *   **Sensors**:
        *   Fermion MEMS Smoke Sensor (Analog)
        *   Fermion MEMS VOC Sensor (Analog)
        *   Fermion MEMS CO Sensor (Analog)
        *   Analog Flame Sensor (Analog)
        *   Fermion AHT20 Temp/Humidity Sensor (I2C)
2.  **Data Layer**:
    *   **Direct Streaming**: Live sensor data forwarded to Edge Impulse Studio via `edge-impulse-data-forwarder`.
    *   **Offline Logging**: Python-based logger (`logger/logger.py`) captures serial data to CSVs for batch uploading.
3.  **ML Layer**:
    *   **Edge Impulse**: Platform for DSP (Spectral Analysis) and Neural Network classification.
    *   **Inference**: C++ library exported from Edge Impulse and linked into the Arduino sketch.

## Key Directories & Files

### Documentation
*   `super-guide.md`: **The comprehensive manual.** Contains wiring diagrams, setup steps, code explanations, and thesis resources. Read this first.
*   `READEME.md`: Project introduction and component inventory.
*   `DAY-TO-DAY-SETUP.md`: Quick reference for daily development tasks.

### Firmware (`sketches/`)
*   `fireDetectionSystemV2.ino`: **Main data collection sketch.** Reads all sensors and outputs CSV-formatted data over Serial at 115200 baud.
*   `verify_all_sensors_operational.ino`: Diagnostic sketch to test hardware connections.

### Scripts (`scripts/`)
*   `automated_data_collection/`: Scripts to automate data logging to CSV.
*   `upload_to_edge_impulse/`: Batch upload collected CSVs to Edge Impulse Studio.
*   `connect-arduino/`: Helper to attach Arduino USB to WSL2 instances.
*   `check-edge-impulse-env/`: Validates local environment dependencies.

### Python Tools (`logger/`)
*   `logger.py`: Custom script to read serial data and save to CSV. Used by the automation scripts.

### Data (`data/`)
*   `strategy.md`: Guide on how to collect balanced datasets (Idle, Fire, Noise).
*   Contains organized subdirectories of collected sensor data (CSV format).

## Workflows

### 1. Data Collection (Live)
1.  Upload `sketches/fireDetectionSystemV2.ino` to Arduino.
2.  Close Arduino Serial Monitor.
3.  Run `edge-impulse-data-forwarder` in terminal.
4.  Follow prompts to connect to your Edge Impulse project.

### 2. Data Collection (Offline/Batched)
1.  Upload `sketches/fireDetectionSystemV2.ino`.
2.  Run `logger/logger.py` (or wrapper scripts) to save data to local CSVs.
3.  Run `scripts/upload_to_edge_impulse/upload_to_edge_impulse.sh` to push files to the cloud.

### 3. Deployment
1.  Train model in Edge Impulse Studio.
2.  Export as **Arduino Library**.
3.  Install library in Arduino IDE.
4.  Create/Upload inference sketch (see `super-guide.md` section 8).

## Development Conventions
*   **Baud Rate**: `115200` is the standard for all serial communication in this project.
*   **Data Format**: `smoke, flame, temp, hum, voc, co` (comma-separated, no spaces, newline terminated).
*   **Sampling Rate**: 10Hz (100ms interval).
*   **Platform**: Linux is the primary dev environment (native or WSL2).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jimzord12)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jimzord12)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
