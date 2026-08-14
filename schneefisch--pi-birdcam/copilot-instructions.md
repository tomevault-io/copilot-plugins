## pi-birdcam

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Raspberry Pi bird camera (`meisencam`) with motion detection and Nextcloud WebDAV upload. Runs on a Raspberry Pi 3+ with
IR camera, triggered every x seconds via cron. Images are stored on a RAM disk (`/mnt/ramdisk`) to avoid SD card wear.

## Commands

```bash
# Install dependencies
uv sync

# Run the capture cycle
python -m meisencam

# Run tests
pytest

# Run a single test file
pytest tests/test_motion.py
pytest tests/test_upload.py
```

## Architecture

The project follows a single-cycle design — each invocation captures one image, checks for motion, and uploads if motion
is detected. Cron handles scheduling (every 15 seconds).

**Modules (`src/meisencam/`):**

- **`__main__.py`** — Entry point. Orchestrates: capture → motion detect → upload → log. Constants:
  `RAMDISK=/mnt/ramdisk`
- **`config.py`** — Centralised configuration loaded from environment / `.env` file. Camera, motion, and upload settings.
- **`camera.py`** — `MeisenCamera` class wrapping `picamera2`. Resolution 1920x1080 with tuned exposure/gain/color
  settings for IR camera
- **`motion.py`** — `detect_motion()` uses per-pixel thresholding on a 64x48 grayscale grid with Gaussian blur. Returns
  percentage of changed pixels (0-100); above threshold triggers upload. Reference image only updated on motion or max age.
- **`upload.py`** — `upload_image()` sends image via WebDAV PUT to Nextcloud public share. Skips upload when `mode < 1`

## Runtime Environment

- **Target:** Raspberry Pi 3+ with Raspbian Light, Camera V2.1 (IR)
- **SSH:** `ssh froeser@192.168.178.40` — project at `~/pi-birdcam`, `uv` at `~/.local/bin/uv`
- **Python:** 3.11
- **RAM disk:** `tmpfs /mnt/ramdisk tmpfs nodev,nosuid,size=64M 0 0` in `/etc/fstab`
- **`picamera2`** is Linux-only — tests mock it; development on macOS uses `uv sync` which skips the platform-specific
  dependency

## Key Dependencies

- `picamera2` — camera control (Linux/Pi only)
- `Pillow` — image processing for motion detection
- `requests` — WebDAV upload
- `pytest` — testing

---
> Source: [schneefisch/pi-birdcam](https://github.com/schneefisch/pi-birdcam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
