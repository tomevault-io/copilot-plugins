## open-cv

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Computer vision portfolio with three projects and standalone learning scripts:

- **scripts/** — standalone OpenCV/YOLO experiments
- **projects/robovision/** — depth-aware 3D perception pipeline (Faster R-CNN + MiDaS)
- **projects/adapttrack/** — uncertainty-aware multi-object tracking (ByteTrack + MC Dropout)
- **projects/synthforge/** — synthetic data generation & active learning pipeline

## Setup

```bash
source .venv/Scripts/activate  # Windows Git Bash
pip install -r requirements.txt
```

## Run Commands

```bash
# Scripts (from repo root)
python scripts/edge_detection.py
python scripts/object_detection.py
python scripts/live_camera_detection.py
python scripts/motion_detection.py

# RoboVision
cd projects/robovision && python main.py --image assets/sample.jpg
cd projects/robovision && python main.py --webcam

# AdaptTrack
cd projects/adapttrack && python main.py --demo
cd projects/adapttrack && python main.py --webcam --log tracking_log.jsonl
cd projects/adapttrack && python main.py --dashboard

# SynthForge
cd projects/synthforge && python main.py demo
cd projects/synthforge && python main.py generate --n-images 100
cd projects/synthforge && python main.py augment --source output/synthetic
```

## Architecture

Each project has its own `src/` with modular components and a `main.py` entry point.

- Model weights (.pt) are gitignored — download separately via ultralytics or torch.hub
- Each project has its own `requirements.txt` plus a root-level consolidated one
- Outputs and datasets are gitignored

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sahilmaniyar888) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
