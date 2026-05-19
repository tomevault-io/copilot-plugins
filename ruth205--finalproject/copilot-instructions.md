## finalproject

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Prosthetic Arm Project — Claude Code Context

## Stack
- Firmware: ESP32 / PlatformIO / C++
- ML: Python 3.11, snnTorch, scikit-learn, numpy
- Power logging: Python + INA219 over I2C

## Conventions
- C++: follow Arduino style, use FreeRTOS tasks
- Python: type hints, docstrings on all functions
- Commit style: conventional commits (feat:, fix:, docs:)

## Key files
- firmware/include/config.h — all pin definitions, change here first
- ml/src/snn/ — the neuromorphic pipeline, our novel contribution
- firmware/src/main.cpp — FreeRTOS task structure

# Neuromorphic Prosthetic Control System

![CI](https://github.com/YOUR_USERNAME/prosthetic-arm/actions/workflows/ci.yml/badge.svg)
![Python](https://img.shields.io/badge/python-3.11-blue)
![Platform](https://img.shields.io/badge/platform-ESP32-red)
![Framework](https://img.shields.io/badge/framework-PlatformIO-orange)
![License](https://img.shields.io/badge/license-Academic-lightgrey)

> Real-time EMG-driven prosthetic hand control using event-driven Spiking Neural Networks — benchmarked against classical ML and deep learning baselines on latency, accuracy, and power draw.

**Final Year Engineering Project**

---

## Overview

This system captures surface EMG signals from the forearm, classifies intended hand gestures in real time using three competing ML pipelines, and drives an [InMoov 3D-printed right hand and forearm](https://inmoov.fr/hand-and-forarm/) via six servo motors. The core research contribution is a neuromorphic SNN classifier that operates on spike-encoded EMG features, targeting lower inference latency and power consumption than conventional approaches.

```
Forearm EMG  →  ESP32 ADC  →  Feature Extraction  →  Classifier  →  Servo Control
     ↑                                                                      ↓
  MyoWare 2.0                  RMS · MAV · ZCR · WL         InMoov right hand + forearm
  4 channels                   4th-order Butterworth         5× finger servos + 1× wrist
  1 kHz sampling               200 ms window / 50% overlap   FSR force feedback · INA219
```

---

## ML Pipeline Comparison

| Pipeline | Classifier | Target latency | Key advantage |
|---|---|---|---|
| Baseline | SVM · Random Forest | < 100 ms | Interpretable, fast to train |
| Deep | CNN · MLP (PyTorch) | < 80 ms | End-to-end feature learning |
| **Neuromorphic** | **Spiking Neural Net** | **< 50 ms** | **Event-driven, low power** |

All three pipelines are trained on the same labelled EMG dataset and evaluated identically — same test split, same inference hardware, same power measurement setup.

---

## Hardware

| Component | Part | Role | Interface |
|---|---|---|---|
| EMG sensors | MyoWare 2.0 ×4 | Forearm muscle signals | Analog → ADC1 |
| IMU | MPU-6050 | Wrist orientation | I2C `0x68` |
| Microcontroller | ESP32-WROOM-32 | Sampling · inference · control | — |
| Power monitor (logic) | INA219 | ESP32 + sensor rail power | I2C `0x40` |
| Power monitor (servo) | INA219 | Servo rail power | I2C `0x41` |
| Prosthetic hand | InMoov right hand (3D printed) | 5 independently actuated fingers | PWM |
| Prosthetic forearm | InMoov forearm (3D printed) | Houses 5× finger servos + 1× wrist servo | PWM |
| Finger servos | MG996R ×5 | Independent finger flexion via fishing line tendons | PWM |
| Wrist servo | MG996R ×1 | Wrist rotation (0–180°) | PWM |
| Force sensors | FSR 402 ×4 | Fingertip contact feedback | Analog → ADC1 |
| Co-processor *(opt.)* | Raspberry Pi 4 | Heavy ML · data logging | UART / WiFi |

### InMoov Hand & Forearm

The physical prosthetic is the open-source [InMoov right hand and forearm](https://inmoov.fr/hand-and-forarm/) designed by Gaël Langevin — a life-sized, 3D-printable robotic hand. Key mechanical details:

- **5 independently actuated fingers** — each driven by a dedicated servo motor housed in the forearm, connected to the finger via braided fishing line tendon (200 lb breaking strength) routed through transparent tubing
- **1 wrist servo** — controls wrist rotation, centred at 90° (flat position)
- **Tendon return springs** — provide tension for realistic finger extension
- **Servo bed** — printed enclosure inside the forearm that mounts all 6 servos
- **STL files** — [InMoov right hand](https://inmoov.fr/inmoov-stl-parts-viewer/?bodyparts=Right-Hand) · [Forearm assembly](https://inmoov.fr/hand-and-forarm/)
- **Print settings** — finger parts at 1.5 mm wall thickness; PLA or ABS; parts can be bonded with acetone, epoxy, or two-part plastic adhesive

---

## Repo Structure

```
prosthetic-arm/
│
├── firmware/                    # ESP32 C++ — PlatformIO project
│   ├── platformio.ini           # Board config, libs, 4 build environments
│   ├── include/
│   │   ├── config.h             # ← ALL pin/rate/address constants live here
│   │   ├── emg.h                # EMGReader — ADC ISR, filter, features
│   │   ├── imu.h                # IMUReader — MPU-6050, orientation
│   │   ├── power_monitor.h      # PowerMonitor — dual INA219
│   │   ├── fsr.h                # FSRReader — force sensing + safety
│   │   ├── servo_control.h      # ServoController — gesture → PWM
│   │   ├── classifier.h         # Classifier — SVM / SNN inference
│   │   ├── logger.h             # Logger — CSV serial output
│   │   └── snn_weights.h        # ← generated by ml/src/snn/export_weights.py
│   └── src/
│       ├── main.cpp             # FreeRTOS task setup
│       ├── emg.cpp
│       ├── imu.cpp
│       ├── power_monitor.cpp
│       ├── fsr.cpp
│       ├── servo_control.cpp
│       ├── classifier.cpp
│       └── logger.cpp
│
├── ml/                          # Python ML pipeline
│   ├── src/
│   │   ├── preprocessing/
│   │   │   ├── design_filter.py     # Butterworth coefficient generator
│   │   │   ├── collect_data.py      # Serial reader + gesture labeller
│   │   │   ├── extract_features.py  # Windowing + RMS/MAV/ZCR/WL
│   │   │   └── visualise_emg.py     # Signal quality plots
│   │   ├── classical/
│   │   │   ├── train_svm.py
│   │   │   ├── train_rf.py
│   │   │   └── train_mlp.py
│   │   ├── snn/
│   │   │   ├── encode.py            # Poisson spike encoder
│   │   │   ├── model.py             # snnTorch LIF network
│   │   │   ├── train_snn.py         # BPTT training loop
│   │   │   └── export_weights.py    # → firmware/include/snn_weights.h
│   │   └── evaluation/
│   │       ├── benchmark.py         # 100-trial latency + accuracy comparison
│   │       ├── plot_results.py      # Publication-quality figures
│   │       └── confusion_matrix.py
│   ├── data/
│   │   ├── raw/                 # gitignored — EMG session CSVs
│   │   ├── processed/           # gitignored — feature matrices
│   │   └── models/              # gitignored — .pkl / .pt checkpoints
│   └── notebooks/               # Exploratory Jupyter notebooks
│
├── power/
│   ├── src/
│   │   ├── log_power.py         # INA219 serial → timestamped CSV
│   │   └── analyse_power.py     # Power draw plots + summary stats
│   └── logs/                    # gitignored — session power logs
│
├── hardware/
│   ├── bom/bom.csv              # Bill of materials
│   └── schematics/              # KiCad / Fritzing wiring diagrams
│
├── docs/
│   ├── figures/                 # Generated plots for report
│   ├── calibration.md           # EMG placement + sensor calibration steps
│   └── gesture_definitions.md   # Gesture specs + servo angles
│
├── .github/workflows/ci.yml     # lint · pytest · PlatformIO compile
├── requirements.txt             # Python deps (install from repo root)
├── setup.cfg                    # pytest · flake8 · black config
├── CLAUDE.md                    # Claude Code project context
└── .gitignore
```

---

## Quickstart

### 1 — Clone and set up Python

```bash
git clone https://github.com/YOUR_USERNAME/prosthetic-arm.git
cd prosthetic-arm

python -m venv .venv
source .venv/bin/activate

# Install PyTorch CPU build first
pip install torch --index-url https://download.pytorch.org/whl/cpu

# Install everything else
pip install -r requirements.txt
```

### 2 — Flash firmware to ESP32

```bash
cd firmware

# Production build
pio run -e esp32dev -t upload

# Open serial monitor
pio device monitor
```

> **Build environments**
> | Environment | Use case |
> |---|---|
> | `esp32dev` | Day-to-day development |
> | `esp32dev_debug` | Crash debugging — verbose logs + stack canaries |
> | `esp32dev_eval` | Benchmarking — serial logging OFF for clean power readings |
> | `native` | Unit tests on Mac/Linux, no hardware needed |

### 3 — Collect training data

Connect ESP32 via USB, arm relaxed:

```bash
python ml/src/preprocessing/collect_data.py --port /dev/ttyUSB0 --output ml/data/raw/session_001.csv
```

Press keys `0`–`5` to label gestures in real time. Aim for 50+ repetitions per gesture per session.

### 4 — Extract features and train

```bash
# Feature extraction
python ml/src/preprocessing/extract_features.py

# Train all classifiers
python ml/src/classical/train_svm.py
python ml/src/classical/train_rf.py
python ml/src/classical/train_mlp.py
python ml/src/snn/train_snn.py

# Export SNN weights → firmware header
python ml/src/snn/export_weights.py
```

### 5 — Run benchmark evaluation

```bash
python ml/src/evaluation/benchmark.py
python ml/src/evaluation/plot_results.py
```

### 6 — Log power consumption

```bash
# With ESP32 running esp32dev_eval build:
python power/src/log_power.py --duration 120 --output power/logs/snn_eval.csv
python power/src/analyse_power.py --input power/logs/snn_eval.csv
```

---

## Gesture Classes

| ID | Gesture | Primary muscles | InMoov servo mapping |
|---|---|---|---|
| 0 | Rest | — | All fingers open, wrist neutral |
| 1 | Fist | Flexor Digitorum Superficialis | All 5 finger servos contract |
| 2 | Open hand | Extensor Digitorum | All 5 finger servos extend |
| 3 | Pinch | Flexor Pollicis Longus | Thumb + index servos contract |
| 4 | Point | Extensor Indicis | Index extends, remaining fingers contract |
| 5 | Wrist flexion | Flexor Carpi Radialis | Wrist servo rotates from 90° |

---

## Signal Processing

EMG signals are conditioned through a 4th-order Butterworth bandpass filter (20–500 Hz) implemented as two cascaded biquad sections running in the ESP32 timer ISR at 1 kHz. Feature extraction runs over a 200 ms sliding window with 50% overlap, producing a 24-element feature vector per inference step:

```
[RMS, MAV, ZCR, WL] × 4 EMG channels  =  16 features
[roll, pitch, yaw, ax, ay, az, gx, gy] =   8 IMU features
                                            ─────────────
                                            24 total
```

To regenerate filter coefficients or change cutoffs:

```bash
python ml/src/preprocessing/design_filter.py
```

---

## Results

*To be completed after evaluation.*

| Metric | SVM | Random Forest | MLP | SNN |
|---|---|---|---|---|
| Accuracy (%) | — | — | — | — |
| Inference latency (ms) | — | — | — | — |
| Logic rail power (mW) | — | — | — | — |
| Servo rail power (mW) | — | — | — | — |
| Parameters | — | — | — | — |

---

## Development

```bash
# Run unit tests (no hardware needed)
pytest

# Lint
flake8 ml/ power/

# Compile all firmware environments
cd firmware && pio run

# Verify filter design
python ml/src/preprocessing/design_filter.py
```

CI runs on every push — lint, pytest, and PlatformIO compile check for all three firmware environments.

---

## Key Design Decisions

**Why InMoov?** The InMoov right hand and forearm is a well-documented open-source design with independent per-finger actuation via tendon-driven servos. All 6 servos (5 fingers + wrist) are housed in the forearm, keeping the hand profile slim and anatomically realistic. The fishing-line tendon system allows fine-grained gesture reproduction from PWM commands.

**Why ESP32?** Dual-core allows the sensor/inference pipeline to run on Core 1 uninterrupted while servo PWM and logging run on Core 0. The hardware timer ISR guarantees 1 kHz sampling jitter < 1 µs.

**Why two INA219s?** Separating the logic rail (ESP32 + sensors, ~150 mW) from the servo rail (~1–5 W) allows the power benchmark to isolate compute cost from actuation cost — essential for a fair comparison between classifiers.

**Why snnTorch?** It wraps PyTorch and supports BPTT with surrogate gradients natively, making SNN training no harder than a standard RNN. The trained weights export cleanly to a C header via `export_weights.py`.

**Why `#ifndef NATIVE_BUILD`?** All feature extraction and ML logic compiles on Mac/Linux for unit testing via the PlatformIO `native` environment. Hardware-specific code (ADC, FreeRTOS, I2C) is guarded behind this macro so CI never needs an ESP32.

---

## License

Academic project — all rights reserved. Not licensed for commercial use.

---
> Source: [ruth205/FinalProject](https://github.com/ruth205/FinalProject) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
