## hummingbirdfeedercam

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HummingBirdFeederCam — a Raspberry Pi + USB camera system that detects hummingbirds using Azure Custom Vision and posts video clips (with audio!) to the "Backyard Hummers" Facebook page. Runs during daytime hours only (sunrise to sunset).

## Development Setup

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

Note: On the Pi, install PortAudio for audio capture: `sudo apt install libportaudio2`. On dev machines, camera and audio functions must be mocked.

## Commands

```bash
python main.py                              # Run the application
pytest                                       # Run all tests
pytest tests/test_motion.py::test_name -v   # Run a single test
ruff check .                                 # Lint
ruff format .                                # Format
```

## Architecture

Data flow: `Camera → Motion Detection → Azure Custom Vision → Clip Recording → Azure OpenAI Caption → Facebook Upload`

```
main.py                  # Orchestrator loop: daytime check → capture → detect → record → post
hummingbird/
  config.py              # YAML loader with ${ENV_VAR} substitution for secrets
  scheduler.py           # Sunrise/sunset gating via astral library
  camera.py              # USB camera (OpenCV) with JPEG ring buffer for pre-roll + ffmpeg clip recording
  audio.py               # Microphone capture via sounddevice with timestamped PCM ring buffer
  motion.py              # OpenCV frame differencing pre-filter (saves Azure API costs)
  detector.py            # Azure Custom Vision prediction API client
  captioner.py           # Azure OpenAI GPT caption generator (funny/cheeky "Backyard Hummers" tone)
  recorder.py            # Assembles clips from circular buffer + post-detection recording
  publisher.py           # Facebook Graph API video upload with retry/cooldown
  webserver.py           # MJPEG live stream server (http://<pi-ip>:8080)
```

Key design decisions:
- **USB camera with microphone** — captures audio alongside video for richer Facebook posts
- **Motion pre-filter before Azure calls** — reduces API costs from ~$1700/day to ~$1-4/day
- **JPEG-compressed ring buffer** — ~9 MB for 3s pre-roll vs 243 MB for raw frames
- **ffmpeg muxing** — combines video frames + audio into MP4 clips with sound
- **Synchronous architecture** — single camera on Pi is inherently sequential; async adds no benefit

## Configuration

- `config.yaml` — all settings (copy from `config.example.yaml`)
- `.env` — secrets (`AZURE_PREDICTION_KEY`, `AZURE_OPENAI_KEY`, `FACEBOOK_PAGE_TOKEN`); referenced in config.yaml as `${ENV_VAR}`

## External Service Setup Required

1. **Azure Custom Vision**: Train a classification model at customvision.ai with tags: "hummingbird", "other_bird", "empty_feeder" (~50-100 labeled images)
2. **Azure OpenAI**: Deploy a GPT model (e.g., gpt-4o) — used to generate funny captions for each post
3. **Facebook Page Token**: Create a Facebook App → get long-lived Page Access Token for the "Backyard Hummers" page

## Deployment

Copy `hummingbird.service` to `/etc/systemd/system/` on the Pi and enable it for auto-start on boot.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LeeOtts)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LeeOtts)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
