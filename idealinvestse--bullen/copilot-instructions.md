## bullen

> These guidelines are tailored for the current project state. Follow them when implementing changes with the Windsurf/Cascade agent.


# Windsurf Code Agent Guidelines – Bullen (RPi5 + Audio Injector Octo)

These guidelines are tailored for the current project state. Follow them when implementing changes with the Windsurf/Cascade agent.

## 1) Project summary
- Purpose: Route 6 analog phone inputs to a headset (stereo monitor), with per-channel gain/mute, fast channel switching, per-channel recording, and simple VU meters.
- Platform: Raspberry Pi 5 + Audio Injector Octo (6 in / 8 out). PipeWire with JACK-API (or JACKd2).
- Stack:
  - Python FastAPI + WebSocket UI (`app/server/`)
  - JACK audio engine (`app/engine/audio_engine.py`)
  - Static web UI (`app/ui/index.html`)
  - Config YAML (`config.yaml`), systemd service (`systemd/bullen.service`)

## 2) Repository map (key paths)
- `app/engine/audio_engine.py`: Real-time JACK engine (6 in, 2 out), gain/mute, selected-channel monitor, VU (peak/RMS), per-channel recording.
- `app/server/app.py`: FastAPI API + WS publisher (VU @ ~20 Hz), mounts UI at `/ui`, root redirects to `/ui/`.
- `app/server/main.py`: Builds `AudioEngine` from config and exposes `app` for Uvicorn.
- `app/ui/index.html`: Minimal UI for select/mute/gain and meters.
- `config.yaml`: Runtime configuration (samplerate hints, frames, auto-connect, selected channel, etc.).
- `Bullen.py`: Entrypoint that runs Uvicorn (`app.server.main:app`).
- `systemd/bullen.service`: Autostart service (uses venv if present).
- `requirements.txt`: Python dependencies.

## 3) Runtime and real-time constraints
- JACK callback must stay non-blocking:
  - Do not perform I/O, logging, or heavy Python work inside `AudioEngine._process()`.
  - Use vectorized NumPy operations (no Python per-sample loops).
  - Keep queues small and drop when full. We track `_rec_drop_counts` per channel.
- Latency targets: start with 48 kHz, 128 frames/period, 2 periods (~5.3 ms). Adjust only if XRUNs.
- Auto-connect strategy uses substring matches (`capture_match`, `playback_match`). Ensure they fit the target card/headset names.

## 4) API and UI contract
- Endpoints:
  - `GET /api/state` → full state (SR, blocksize, selected_channel, gains, mutes, VU, recording, drop counts)
  - `POST /api/select/{ch}` → select 1..6
  - `POST /api/gain/{ch}` with `{gain_db}` or `{gain_linear}`
  - `POST /api/mute/{ch}` with `{mute: bool}`
  - `GET /api/config` → effective config
- WebSocket: `/ws/vu` publishes JSON with `vu_peak`, `vu_rms`, `selected_channel`, `mutes`, `gains_db` ~20 Hz.
- UI assumptions: 6 channels; if you change `inputs`, update UI accordingly (or make dynamic).

## 5) Config and environment
- Default load order: `BULLEN_CONFIG` env var → `config.yaml` in repo root → built-in defaults in `app/config.py`.
- Key options in `config.yaml`:
  - `samplerate`, `frames_per_period`, `nperiods` (informational; actual SR/buffer governed by JACK/PipeWire)
  - `inputs`, `outputs` (default 6/2)
  - `record` (bool), `recordings_dir`
  - `auto_connect_capture`, `auto_connect_playback`, `capture_match`, `playback_match`
  - `selected_channel` (1-based)

## 6) Coding and style conventions
- Python ≥ 3.10. Use type hints. Keep imports at top of file.
- Maintain separation of concerns:
  - Real-time DSP/routing only in `AudioEngine`.
  - Control plane, HTTP/WS in server.
  - UI changes in `app/ui/` and keep it lightweight.
- Avoid blocking or heavy allocations in JACK callback. Use `.copy()` only when necessary (e.g., to enqueue for recording).
- When adding dependencies, update `requirements.txt` and document usage in `README.md`.

## 7) Development workflow (for Windsurf/Cascade)
- Reading/analyzing code: open files fully before changes.
- Making changes: NEVER paste big code into chat; use the code edit tools to patch files. Keep patches minimal and with correct context.
- Plan tasks: keep a TODO list and mark items completed as you proceed.
- Running commands: prefer safe, read-only. For apt/pip/systemd or network changes, ask for approval and explain impact.
- Testing locally on Pi:
  - Verify devices with `arecord -l` / `aplay -l`.
  - Run `python3 Bullen.py` and open `http://<Pi-IP>:8000/`.
  - Check XRUNs with `pw-top` or `qpwgraph`.

## 8) Adding features – guidance
- Multi-channel mixing to headset:
  - Add a per-channel `monitor_enable` and compute an L/R mix array in `_process()`.
  - Keep vectorized mixing; avoid Python loops.
- Post-gain recording:
  - Add a config flag (e.g., `record_post_gain`) and enqueue post-gain buffers when true.
- DSP (AGC/limiter/NS/AEC):
  - Use SpeexDSP/WebRTC in a non-RT worker or highly optimized path.
  - Ensure fixed-size buffers and no dynamic allocations in RT.
- Device selection:
  - Extend auto-connect with explicit port names in config; add UI to pick targets; reconnect on change.

## 9) Systemd and deployment
- Service: `systemd/bullen.service` assumes `/home/pi/Bullen`, `User=pi`, and project venv at `.venv`.
- If paths/users differ, update `WorkingDirectory`, `Environment`, and `ExecStart`.
- Use `journalctl -u bullen.service -f` for logs.

## 10) Performance and acceptance criteria
- No audible glitches during rapid channel switching.
- VU updates ~20 Hz; UI remains responsive.
- Sustained recording of all 6 channels without queue overflows (drop counts near zero under expected load).
- Cold boot brings the service up and auto-connects to the correct devices.

## 11) Known limitations (MVP)
- Only one selected channel is monitored to L/R.
- No DSP (AGC/limiter/NS/AEC) yet.
- No authentication; intended for LAN use. If exposing externally, add auth and TLS.

## 12) Safe command examples (Pi)
- Read-only:
  - `arecord -l`, `aplay -l`, `pw-top`, `cat /boot/firmware/config.txt`
- Requires approval (mutating):
  - `sudo apt install ...`, `sudo systemctl ...`, modifying `/boot/firmware/config.txt`

## 13) Code review checklist (agent)
- Audio callback remains non-blocking and vectorized.
- Shared state mutations are protected by `self._lock`.
- No busy loops; WS/VU publisher uses sleep/backoff.
- Config keys documented; sensible defaults preserved.
- UI updates consistent with API responses and channel count.

## 14) Quick reference – endpoints
- `GET /api/state` – current engine state
- `POST /api/select/{1..6}` – select channel
- `POST /api/gain/{1..6}` – body: `{gain_db: number}` or `{gain_linear: number}`
- `POST /api/mute/{1..6}` – body: `{mute: boolean}`
- `GET /api/config` – effective configuration
- UI: `GET /ui/` – control panel

Adhere strictly to these guidelines to maintain real-time safety, clarity, and reliability of the system.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/idealinvestse) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
