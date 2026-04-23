## analyze-video

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Surveillance video analysis system ("Споглядайко") that monitors a folder for new `.mp4` files, detects motion and objects, generates AI descriptions via Google Gemini, and sends results to Telegram. Designed for a residential gate/entrance camera with gate crossing detection, person re-identification, and optional Tesla integration.

Companion Android dashboard app: [analyze-video-dashboard](https://github.com/alivespirit/analyze-video-dashboard) (Jetpack Compose, consumes the JSON API from `tools/log_dashboard/`).

## Running the Application

```bash
# Setup
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# Run (requires .env with GEMINI_API_KEY, TELEGRAM_TOKEN, TELEGRAM_CHAT_ID, VIDEO_FOLDER)
python main.py
```

The app auto-restarts when any `.py` file is modified (via watchdog + `os.execv`).

## Testing a Single Video

```bash
python tools/run_detect_motion.py /path/to/video.mp4 [output_dir] [--log-level DEBUG]
```

There is no automated test suite. Validation is done via `tools/validate_log_replay.py` for batch re-analysis.

## Architecture

**Pipeline**: File monitoring → Motion detection (local or remote worker) → AI analysis → Telegram notification

**Dual-executor pattern** (critical design choice):
- `motion_executor`: ThreadPoolExecutor(max_workers=1) — CPU-bound local motion detection runs serially
- `io_executor`: ThreadPoolExecutor(max_workers=4) — I/O-bound Gemini/Telegram calls run in parallel
- Remote worker dispatch is fully async (no executor slot consumed while waiting on HTTP)
- Both executors are driven by a single asyncio event loop in `main.py`

### Core Modules

| Module | Role |
|--------|------|
| `main.py` | Entry point, asyncio orchestration, watchdog file monitoring, Telegram bot setup, processing ledger, auto-restart, retention cleanup, Tesla SoC scheduler |
| `detect_motion.py` | Core vision: background subtraction, YOLO tracking (OpenVINO), gate crossing detection, ReID triggering, highlight clip generation, car speed estimation |
| `analyze_video.py` | Gemini API calls with dynamic model selection (time-based Pro vs Flash), fallback chains, prompt loading from `config/prompt.txt` |
| `telegram_notification.py` | Message delivery with retry logic, grouped notifications, inline callbacks for full video, media validation. Respects `KEEP_HIGHLIGHTS_CLIPS` for clip cleanup. |
| `person_id.py` | Intel OpenVINO ReID model, gallery embedding with disk caching (keyed by gallery path + model path), negative gallery support, cosine similarity matching |
| `path_utils.py` | Timestamp extraction from video filenames |
| `worker/client.py` | Master-side worker dispatch: async HTTP, health check with caching, local fallback (lazy YOLO load), log replay with `[W]` tagging, Wake-on-LAN |
| `worker/server.py` | FastAPI worker server: path translation (Windows↔Linux), video copy, detect_motion, result copy back to CIFS, gallery cache pre-warming |
| `tools/log_dashboard/app.py` | FastAPI dashboard: HTML log viewer + JSON API for Android app (per-video summaries, stats, monitoring, events, image/clip serving, ReID gallery management) |

### Data Flow

1. `main.py` watchdog detects new `.mp4` → checks worker availability
2. If worker available: async HTTP dispatch to `worker/server.py`; otherwise queues to `motion_executor` locally
3. `detect_motion.py` returns result dict with status, clip path, person/car counts, crossing info, ReID results
4. `analyze_video.py` decides whether to call Gemini based on motion status and time of day
5. `telegram_notification.py` sends animation/photo/text with retry and grouping logic

### Directory Structure

```
analyze-video/
├── main.py, detect_motion.py, analyze_video.py   # core pipeline
├── telegram_notification.py, person_id.py        # core pipeline
├── path_utils.py                                 # utility
├── config/                                       # runtime configuration files
│   ├── roi.json, roi-1080p.json, roi-4k.json     # ROI polygons
│   ├── tracker.yaml                              # ByteTrack/BoTSORT config
│   ├── gemini_models.env                         # Gemini model names (hot-reload)
│   └── prompt.txt                               # Gemini prompt (Ukrainian)
├── worker/                                       # remote worker package
│   ├── server.py                                 # FastAPI worker server
│   ├── client.py                                 # master-side dispatch client + WOL
│   ├── worker.service                            # systemd unit
│   └── requirements.txt                          # worker-only dependencies
├── tools/                                        # dev/ops utilities
│   └── log_dashboard/                            # HTML dashboard + JSON API
│       ├── app.py                                # FastAPI app (HTML routes + /api/* JSON routes)
│       ├── templates/                            # Jinja2 HTML templates
│       └── static/                               # CSS
├── models/                                       # YOLO + ReID model files
└── temp/                                         # runtime temp files, caches, and daily dirs
    ├── processing_ledger.json                    # file processing status tracker
    ├── tesla_soc.txt                             # Tesla SoC cache
    └── YYYYMMDD/                                 # daily dirs with highlight clips, frames, ReID crops
```

### Key Configuration

- **`.env`**: All secrets and runtime config (API keys, paths, thresholds)
- **`config/gemini_models.env`**: Gemini model names, re-read on every call (no restart needed)
- **`config/roi-1080p.json` / `config/roi-4k.json`**: Resolution-specific ROI polygons for motion detection and tracking
- **`config/tracker.yaml`**: ByteTrack/BoTSORT tracking parameters
- **`config/prompt.txt`**: Gemini prompt (in Ukrainian)

### Resolution Handling

`detect_motion.py` has a `RES_CONFIGS` dict with per-resolution settings (1080p, 4K). ROI configs are loaded from resolution-specific JSON files in `config/` with fallback to `config/roi.json`. 4K frames are downscaled to 1080p for highlight output. Insignificant/no_person frames are saved at original resolution (4K).

### Remote Worker

Motion detection can be offloaded to a worker machine over HTTP (`worker/server.py`, FastAPI). The master dispatches asynchronously via `worker/client.py` and falls back to local processing if the worker is unavailable. Worker logs are replayed into the master log with `[W]` inserted after the `[filename]` bracket. See `worker/README.md` for setup.

Key worker env vars: `WORKER_ENABLED`, `WORKER_URL`, `WORKER_TIMEOUT`, `WORKER_MIN_BATTERY`.

The worker `/health` endpoint reports: status, active/max tasks, battery percent, load averages (1m/5m/15m), memory usage, and CPU temperature (Package id 0 from coretemp).

### Wake-on-LAN

The master can automatically wake the worker when power is restored. When the worker health check fails and the master is plugged in (AC power), a WOL magic packet is sent via the configured network interface with a 5-minute cooldown between attempts. Implemented in `worker/client.py`.

Key env vars: `WORKER_WAKE_ON_LAN` (bool, default false), `WORKER_WAKE_ON_LAN_MAC` (MAC address), `WORKER_WAKE_ON_LAN_IFACE_IP` (bind IP for WOL broadcast, default `10.0.0.1`).

### Highlight Clips and Daily Directories

Highlight clips are saved to daily subdirectories under `TEMP_DIR` (`temp/YYYYMMDD/<video>.mp4`). The `KEEP_HIGHLIGHTS_CLIPS` env var (default `true`) controls whether clips are preserved after Telegram send — when true, clips remain for viewing in the Android dashboard and are cleaned up by the daily directory retention process. Both `main.py` and `telegram_notification.py` check this setting.

Insignificant/no_person frames and ReID crops are also saved to these daily directories:
- Frames: `{hour}H{video_stem}_{tag}_{frameIdx}.jpg` (tag = `insignificant` or `no_person`)
- ReID crops: `{video_stem}_reid_best{N}.jpg`

### Tesla SoC

Tesla State of Charge is fetched by a periodic scheduler in `main.py` (`tesla_soc_scheduler`) and written to a cache file (`TESLA_SOC_FILE`, default `temp/tesla_soc.txt`). `detect_motion.py` only reads the cache — it has no Tesla credentials or API calls.

### ReID Gallery Cache

`person_id.py` caches gallery embeddings as NPZ files in `REID_CACHE_DIR` (default `temp/`). The cache filename includes a hash of both the gallery path and the model path, so master and worker (which use different ReID models) maintain separate cache files even when `REID_CACHE_DIR` points to shared NAS storage. The worker pre-warms its cache at startup and periodically via a background loop in `worker/server.py` (`GALLERY_PRECHECK_INTERVAL`, default 60 s).

### Restart Recovery

A JSON ledger (`temp/processing_ledger.json`) tracks file processing status. After restart, files within `RESTART_RECOVERY_WINDOW_SECONDS` (default 180s) are re-queued.

### Fast Processing Mode

When motion queue backlog exceeds a threshold, adaptive frame skipping kicks in — controlled by `FAST_MOTION_STRIDE`, `FAST_TRACK_FULL_UNTIL_SECONDS`, `FAST_TRACK_SKIP_FROM_SECONDS` env vars.

## Log Dashboard

The log dashboard (`tools/log_dashboard/app.py`) serves two roles:
1. **HTML dashboard** — browser-based log viewer with per-day insights, filters, charts, and inline video player
2. **JSON API** — consumed by the Android dashboard app

Enable via `ENABLE_LOG_DASHBOARD=true`. Default port: `8192`.

### JSON API endpoints

| Endpoint | Description |
|----------|-------------|
| `GET /api/days` | List of available log days (YYYY-MM-DD) |
| `GET /api/today/videos?day=` | Per-video summary (status, gate, ReID, processing time, `has_frames` indicator) |
| `GET /api/today/video/{basename}/logs?day=` | Log entries for a specific video |
| `GET /api/today/video/{basename}/reid-crops` | ReID crop image URLs (from TEMP_DIR daily dirs) |
| `GET /api/today/video/{basename}/frames` | Insignificant/no_person frame URLs (from TEMP_DIR daily dirs) |
| `GET /api/today/video/{basename}/highlight` | Highlight clip URL if available (from TEMP_DIR daily dirs) |
| `GET /api/today/gate-crossings?day=` | Videos with ReID crops: basename, time, direction, status, scores, crop URLs |
| `GET /api/today/stats?day=` | Aggregated stats (status counts, gate counts, processing times, away/back intervals) |
| `GET /api/stats/overall` | Overall stats with per-day data, events heatmap, weekday heatmap |
| `GET /api/monitoring` | Master CPU/RAM/battery + worker health proxy + recent processing ledger |
| `GET /api/events/latest?since=` | Away/back events with current home/away status (for notifications) |
| `POST /api/reid/copy` | Copy ReID crop to positive or negative gallery |
| `GET /api/image/{basename}` | Serve images (crops, frames) from TEMP_DIR or VIDEO_FOLDER |
| `GET /api/highlight/{basename}` | Serve highlight clips from TEMP_DIR daily dirs |

All `?day=` parameters default to today (or latest available day). API responses use per-day caching (30s TTL, max 7 days in memory). The `_walk_daily_dirs()` helper restricts file searches to `YYYYMMDD`-named subdirectories of `TEMP_DIR` to avoid scanning unrelated folders (e.g., `training/`).

The `has_frames` field in `/api/today/videos` is derived from "Saved ... frame to" log messages via `collect_metrics()`, avoiding filesystem scans during video list construction.

Worker health is proxied via `_get_worker_health()` with 30s cache. When the worker is configured (`WORKER_URL` set) but unreachable, returns `{"status": "offline"}` instead of `null`.

`/api/monitoring` reads master stats from `psutil` — `cpu_percent(interval=None)` (non-blocking, 10s cache), `virtual_memory()`, `sensors_battery()`.

## Language Note

User-facing messages, Telegram captions, and the Gemini prompt are in Ukrainian.

## Tools Directory

- `tools/run_detect_motion.py` — Single video analysis CLI
- `tools/validate_log_replay.py` — Batch re-analysis from logs
- `tools/reid_gallery_dedupe.py` — Find duplicate ReID gallery images
- `tools/finetuning/` — YOLO training and export scripts
- `tools/log_dashboard/` — FastAPI web dashboard + JSON API for Android app (enable via `ENABLE_LOG_DASHBOARD=true`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alivespirit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
