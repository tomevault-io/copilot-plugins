## lingo2

> Lingo 2.0 provides **local speech-to-text** via whisper.cpp with two apps:

# Lingo 2.0 - AI Coding Instructions

## Project Overview
Lingo 2.0 provides **local speech-to-text** via whisper.cpp with two apps:
1. **web-app/** - Browser-based TTS/STT with FastAPI backend (port 8009)
2. **gtk-app/** - System-wide voice typing for Linux (types into any focused app)

**Philosophy**: Framework-free. No React/Vue/build systems. Vanilla HTML/CSS/JS + Python.

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ web-app (Browser)                    │ gtk-app (Linux Desktop)              │
│ lingo.html/js/css                    │ voice_typer.py (GTK4)                │
│     │                                │     │                                │
│     ▼                                │     ▼                                │
│ FastAPI (whisper_server.py:8009)     │ sounddevice → whisper-cli            │
│     │                                │     │                                │
│     └──────────┬─────────────────────┴─────┘                                │
│                ▼                                                            │
│          whisper-model/whisper.cpp/build/bin/whisper-cli                    │
│          whisper-model/whisper.cpp/models/ggml-base.en.bin                  │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Developer Commands

```bash
./setup-whisper.sh      # First-time: builds whisper.cpp, downloads base.en model

# Web app
cd web-app && ./run.sh  # Start server → http://localhost:8009/lingo.html
./kill.sh               # Stop server

# GTK app (system-wide voice typing)
cd gtk-app && ./run.sh  # Launch floating mic button
```

## Code Conventions

### Web Frontend (web-app/lingo.js)
- **Section markers**: `// ============` blocks organize code (TTS State, STT State, etc.)
- **Button sync**: `updateReadButton()` / `updateMicButton()` keep UI in sync with state
- **Storage keys**: `tts_` prefix or `*_v1` suffix for localStorage versioning
- **Status bar**: All ops call `setStatus()` for user feedback

### Python Backend (whisper_server.py, voice_typer.py)
- **Paths**: Always use `Path` from pathlib, relative to `SCRIPT_DIR`
- **Temp files**: UUID-prefixed, cleaned in `finally` block
- **Whisper paths** (relative to each app):
  - Binary: `../whisper-model/whisper.cpp/build/bin/whisper-cli`
  - Model: `../whisper-model/whisper.cpp/models/ggml-base.en.bin`

### GTK App Specifics (gtk-app/voice_typer.py)
- **Audio pipeline**: sounddevice (48kHz) → resample to 16kHz → normalize → whisper-cli
- **Keyboard injection**: Uses XDG Remote Desktop Portal (Wayland-safe)
- **Logging**: Writes to `gtk-app/voice_typer.log` (overwritten each run)
- **Config file**: `~/.config/lingo-gtk.yaml` stores user preferences (microphone selection)
- **Device selection**: GUI dropdown lets user pick microphone; saved to config file

## Silence Detection Config

Both apps use similar silence detection (adjust for your mic):
```python
# gtk-app/voice_typer.py (quieter USB mics)
SILENCE_THRESHOLD = 0.002   # RMS threshold
SILENCE_DURATION_S = 1.0    # Seconds of silence → transcribe
MIN_AUDIO_DURATION_S = 0.5  # Skip very short clips

# web-app/lingo.js (browser)
const SILENCE_THRESHOLD = 0.01;
const SILENCE_DURATION_MS = 1000;
```

## Common Modifications

**Change Whisper model**: Update `WHISPER_MODEL` in whisper_server.py AND voice_typer.py, plus model download in setup-whisper.sh

**Add web keyboard shortcut** (web-app/lingo.js ~line 640):
```javascript
if ((evt.ctrlKey || evt.metaKey) && evt.key.toLowerCase() === "x") {
  evt.preventDefault();
  // action
}
```

**Add API endpoint**: Insert before static files mount in whisper_server.py (line ~208)

**Tune for quiet mics**: Lower `SILENCE_THRESHOLD`, check RMS in gtk-app log

## Dependencies

**Shared** (both apps): `ffmpeg`, whisper.cpp (built via `./setup-whisper.sh`)

**web-app**: `fastapi`, `uvicorn`, `python-multipart` (auto-installed by run.sh)

**gtk-app** (run `./setup.sh` or manually install):
- System: `python3-gi`, `gir1.2-gtk-4.0`, `portaudio19-dev` (Ubuntu/Debian names)
- Python (via requirements.txt): `sounddevice`, `numpy`
- Keyboard injection: Uses XDG Remote Desktop Portal (via GLib/Gio, included with PyGObject)

---
> Source: [Clay-Ferguson/lingo2](https://github.com/Clay-Ferguson/lingo2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
