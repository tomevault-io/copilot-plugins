## head-tracking-monitor-arm-rp5

> A computer vision system running on a **Raspberry Pi 5** that performs real-time face detection/recognition and controls hardware (GPIO, servos, motors) based on detected faces. Video is streamed via a Flask MJPEG server accessible over the local network.

# Project: Raspberry Pi Face Detection & Control System

## Overview
A computer vision system running on a **Raspberry Pi 5** that performs real-time face detection/recognition and controls hardware (GPIO, servos, motors) based on detected faces. Video is streamed via a Flask MJPEG server accessible over the local network.

## Project Structure

| Folder | Purpose |
|--------|---------|
| `Face Detection/` | Real-time face detection (YOLO, Haar Cascades) |
| `Face Detection/yolo_face/` | Optimised YOLO scripts and models for RPi |
| `Face recognition/` | Face encoding and recognition (dlib / face_recognition) |
| `Server/mjpeg_server/` | Flask MJPEG server — receives frames, broadcasts stream |
| `Servo Testing/` | DC motor / servo hardware testing (HR8833 driver) |
| `toggle_pi_gpio/` | YOLO detection + GPIO toggle (LED on face detected) |

## Network / Hardware Config

| Item | Value |
|------|-------|
| MJPEG input stream | `https://192.168.1.111:8002/mjpeg` |
| Detection output stream | `http://192.168.1.111:8004/stream.mjpeg` |
| Camera fallback IP | `192.168.1.24:8081/video` |
| Flask server ports | 8001 / 8002 |

**GPIO pins:**
- Pin 14 — LED (toggle_pi_gpio)
- Pin 18 — Horizontal servo (PWM)
- Pin 13 — Vertical servo (PWM)
- Pins 20–21 — HR8833 motor driver

## Key Files

| File | What it does |
|------|-------------|
| `Server/mjpeg_server/app.py` | Flask server: `/upload` receives frames, `/mjpeg` broadcasts stream |
| `Face Detection/face_detection_stream.py` | Haar Cascade detection from MJPEG stream |
| `Face Detection/face_detection_servo.py` | Face detection + servo tracking |
| `Face Detection/yolo_face/yolo_face_detect_rpi.py` | YOLO detection optimised for RPi 5 |
| `Face Detection/yolo_face/face_detect_ncnn_optimized.py` | NCNN model for fastest RPi inference |
| `Face Detection/prepare_face_dataset.py` | Auto-labels images → YOLO format dataset |
| `Face recognition/detector.py` | Encode known faces, live recognition from stream |
| `toggle_pi_gpio/toggle_pi_gpio.py` | YOLO + GPIO LED toggle on face in ROI |
| `Servo Testing/hr8833_dc_motor_control.py` | H-bridge motor control via GPIO |

## Models & Data

- YOLO model: `Face Detection/yolo_face/models/yolov8n-face.pt` (6.1 MB, WIDERFace trained)
- Face recognition encodings: `Face recognition/output/encodings.pkl`
- Training images: `Face recognition/training/<person_name>/`
- YOLO dataset: `Face Detection/yolo_face/face_detection_dataset/`

## Virtual Environment

```bash
# Face Detection
cd "Face Detection"
source .venv-gui/bin/activate

# Server
cd Server/mjpeg_server
source .venv/bin/activate
```

## Performance (RPi 5)
- YOLO detection: ~10 FPS, 150–180% CPU

## Data Flow
```
Camera/IP Stream
  → MJPEG Server (app.py)
      → /upload  ← detection scripts push frames
      → /mjpeg   → clients / web UI
  → Face Detection (YOLO / Haar)
      → face in ROI? → toggle GPIO (LED)
      → servo tracking
  → Face Recognition (dlib)
      → identify person from encodings
```

## Dependencies

**Face Detection:** numpy, opencv-python, requests, ultralytics, gpiozero, lgpio
**Server:** flask
**Face Recognition:** dlib==19.24.4, face-recognition==1.3.0, numpy, Pillow, requests, pigpio==1.78

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JBIS1104) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
