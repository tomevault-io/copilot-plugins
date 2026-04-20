## canvas-hsg

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HSG Canvas (Hackerspace.gent Canvas) - a FastAPI application that manages media streaming and playback on a Raspberry Pi. Runs alongside an SRS (Simple Realtime Server) to display streams, images, and YouTube content on a connected display. The Pi acts as both a media player and a stream republisher.

## Development Commands

```bash
# Install dependencies
pip3 install -r requirements.txt

# Run server (development, port 8000)
python3 main.py

# Run server (production, port 8000 — Angie proxies port 80)
python3 main.py --production

# Run via start script (starts Vite + FastAPI in production)
./start.sh

# One-time setup (installs Angie, systemd services, raspotify config)
sudo ./setup.sh

# Run tests
pytest tests/
```

## Architecture

### Application Wiring

`main.py` uses FastAPI's lifespan context manager to initialize everything in order:

1. **DisplayCapabilityDetector** - detects connected display resolution via DRM
2. **WebSocket managers** (3 instances) - for Spotify events, display state, and audio commands
3. **DisplayStack** - core display abstraction (base layer + stack of items)
4. **ChromiumManager** - starts Chromium once at boot, never stops
5. **BackgroundManager** - thin wrapper around DisplayStack
6. **AudioManager** - controls browser `<audio>` via WebSocket (no MPV)
7. **PlaybackManager** - pushes YouTube to DisplayStack (browser renders via IFrame API)
8. **All other managers** - each receives its dependencies via constructor injection
9. **API routes** - each `setup_*_routes()` function creates an `APIRouter` with manager references

Shutdown reverses this order. All managers are global module-level variables set during lifespan.

### Key Directories

- **`managers/`** - All business logic (there is no separate `core/` directory)
- **`models/`** - Pydantic request models (`request_models.py`)
- **`routes.py`** - Single file with 12 `setup_*_routes()` functions, each returning an `APIRouter`
- **`config.py`** - Constants: paths, network/discovery, server ports
- **`config/`** - Deployment config files (Angie, systemd service, raspotify drop-in)
- **`background_engine/`** - Advanced background image generation (layout, components, generators)
- **`tests/`** - pytest tests with pytest-asyncio for async testing
- **`static/`** - CSS/JS for the web interface
- **`index.html`** - Synthwave-themed web control panel (served at `/`)

### Manager Interactions

Managers reference each other for coordinated behavior:
- **PlaybackManager** depends on: DisplayStack, DisplayCapabilityDetector, BackgroundManager, AudioManager
- **ChromecastManager** depends on: AudioManager, PlaybackManager (stops local playback when casting)
- **OutputTargetManager** depends on: AudioManager, PlaybackManager, ChromecastManager (unified target interface)
- **SpotifyManager** depends on: AudioManager (stops audio streams when Spotify plays)
- **BackgroundManager** depends on: DisplayStack, DisplayCapabilityDetector

### Route Pattern

Each route group in `routes.py` follows the same pattern:
```python
def setup_X_routes(manager, ...) -> APIRouter:
    router = APIRouter(prefix="/X", tags=["X"])
    # Endpoint definitions using the injected manager
    return router
```

Routes are included in the app during lifespan startup via `app.include_router()`.

## Configuration (`config.py`)

- **DEFAULT_BACKGROUND_PATH**: Path to default background image
- **DEVICE_NAME**: "HSG Canvas" (used in SSDP/Chromecast discovery)
- **CHROMECAST_CACHE_DURATION**: 24 hours
- **METADATA_UPDATE_INTERVAL**: 15 seconds
- **DEFAULT_PORT / PRODUCTION_PORT**: Both 8000 (Angie proxies port 80)

## Hardware Context

This runs on a Raspberry Pi with:
- Chromium kiosk mode for display output
- PipeWire/PulseAudio for audio (shared with Raspotify service)
- Temperature monitoring via `/sys/class/thermal/thermal_zone0/temp`
- HDMI-CEC for TV power control

## Important Patterns

- **Subprocess isolation**: Chromecast discovery runs in a subprocess to prevent zeroconf file descriptor leaks
- **Audio exclusivity**: Only one audio source at a time (audio stream, Spotify, or video with audio)
- **YouTube via IFrame API**: Video ID extracted by regex, rendered in browser (no yt-dlp)

## Critical Gotchas

### setup.sh overwrites config files
`setup.sh` is a full system provisioning script. Running it **replaces** `/etc/raspotify/conf` and systemd service files. If you add config to those files outside of setup.sh, the next setup.sh run will erase it. **Always update setup.sh when changing deployment config.**

Key things setup.sh must preserve:
- `LIBRESPOT_ONEVENT=/home/hsg/srs_server/raspotify-onevent.sh` in `/etc/raspotify/conf` (without this, Spotify events don't reach FastAPI and the display never updates)
- Raspotify systemd drop-in at `config/raspotify/onevent.conf` (User=hsg, PrivateUsers=false, ProtectHome=false)

### Raspotify + PipeWire/PulseAudio
Raspotify's default systemd sandbox has `PrivateUsers=true` (remaps UIDs, breaking socket auth) and `ProtectHome=true` (blocks `/home/hsg`). Both must be overridden in the drop-in for PulseAudio to work. Symptom: `Audio Sink Error Connection Refused: <PulseAudioSink>`.

### Vite base path + Angie proxy
When proxying a Vite app under a subpath (`/spotify/`), you must:
1. Set `base: '/spotify/'` in `vite.config.js` (so asset URLs get the prefix)
2. Use `proxy_pass http://upstream;` (NO trailing slash) in Angie so the `/spotify/` prefix passes through to Vite
If you strip the prefix (`proxy_pass http://upstream/;` WITH trailing slash), Vite gets `/` but its assets reference `/spotify/...` paths → redirect loop.

### Angie apt repo URL format
The correct URL is `deb https://download.angie.software/angie/debian/12 bookworm main` (note the `/12` version number). The format without the version number returns 404.
- **Background restoration**: After playback/image display ends, background image is automatically restored

## Production Deployment (systemd)

HSG Canvas runs as a systemd service on the Raspberry Pi for automatic startup and management.

### Service Management

```bash
# Start/stop/restart the service
sudo systemctl start hsg-canvas
sudo systemctl stop hsg-canvas
sudo systemctl restart hsg-canvas

# Check service status
systemctl status hsg-canvas

# View service logs
sudo journalctl -u hsg-canvas -f          # Follow logs in real-time
sudo journalctl -u hsg-canvas -n 100      # Last 100 lines
sudo journalctl -u hsg-canvas --since today

# Enable/disable auto-start on boot
sudo systemctl enable hsg-canvas
sudo systemctl disable hsg-canvas
```

### Reverse Proxy (Angie)

Angie reverse proxy on port 80 routes all external traffic:

| URL Path | Backend | Content |
|----------|---------|---------|
| `/` | FastAPI:8000 | Control panel |
| `/spotify/` | Vite:5173 | React display app (HMR works) |
| `/static/*` | FastAPI:8000 | Control panel assets |
| `/ws/*` | FastAPI:8000 | WebSocket endpoints |
| `/docs` | FastAPI:8000 | OpenAPI docs |
| All API routes | FastAPI:8000 | REST API |

Config files (all in repo, symlinked/copied by `setup.sh`):
- `config/angie/hsg-canvas.conf` → `/etc/angie/http.d/hsg-canvas.conf`
- `config/hsg-canvas.service` → `/etc/systemd/system/hsg-canvas.service`
- `config/raspotify/onevent.conf` → `/etc/systemd/system/raspotify.service.d/onevent.conf`

### Service Configuration

The service file is at `config/hsg-canvas.service` (copied to `/etc/systemd/system/` by setup.sh).

**Key Points**:
- Runs as user `hsg` (not root)
- Uses virtual environment via `start.sh`
- Auto-restarts on crash (10 second delay)
- No longer needs `CAP_NET_BIND_SERVICE` (Angie owns port 80, FastAPI listens on 8000)
- Logs to systemd journal

### Updating the Service

After making code changes:

```bash
# Restart the service to pick up changes
sudo systemctl restart hsg-canvas

# If you modified the service file itself
sudo systemctl daemon-reload
sudo systemctl restart hsg-canvas
```

## Web-Based Display System

**Status**: Implemented (February 2026)
**Phase**: 1 - Spotify Now-Playing View

The system uses **Chromium in kiosk mode** for modern, web-based display rendering instead of PIL/FFmpeg image/video generation.

### Architecture

```
Spotify Event (librespot onevent hook)
    ↓  POST /audio/spotify/event
SpotifyManager → broadcasts via WebSocket
    ↓
WebSocketManager → pushes to all clients
    ↓
React App at /spotify/ (via Angie → Vite:5173)
    ↓
Chromium Kiosk Mode (cage Wayland → http://127.0.0.1/spotify/)
    ↓
Physical Display (HDMI output)
```

### Components

#### 1. WebSocket Infrastructure (`managers/websocket_manager.py`)
- Real-time event broadcasting to connected clients
- Endpoint: `ws://localhost/ws/spotify-events`
- Status: `GET /ws/status` → returns active connection count
- Events: `track_changed`, `playback_state`, etc.

#### 2. Chromium Manager (`managers/chromium_manager.py`)
- Launches Chromium browser in full-screen kiosk mode
- Uses Xvfb (`:99`) for headless X11 virtual display
- Auto-detects display resolution from DRM
- Proper process cleanup (SIGTERM → SIGKILL escalation)

**Dependencies**:
```bash
sudo apt-get install xvfb chromium-browser
.venv/bin/pip install jinja2
```

#### 3. Now-Playing Web Interface
- **Template**: `templates/now-playing.html`
- **CSS**: `static/css/now-playing.css` (animations, blur effects)
- **JavaScript**: `static/js/now-playing.js` (WebSocket client)
- **Route**: `GET /now-playing` → renders Jinja2 template

**Features**:
- Blurred album art background
- Google Sans font
- Auto-scrolling for long track names
- Real-time updates via WebSocket
- Reconnection logic

### Display Mode Switching

The system supports multiple **mutually exclusive** display modes:

| Mode | Renderer | Manager | Notes |
|------|----------|---------|-------|
| YouTube Video | Chromium (IFrame API) | PlaybackManager | Browser-rendered |
| Audio Streams | Browser `<audio>` | AudioManager | WebSocket-controlled |
| Static Background | Chromium | BackgroundManager | Default mode |
| Spotify Now-Playing | Chromium (React) | SpotifyManager | Real-time via WebSocket |
| Image Display | Chromium | BackgroundManager | QR codes, etc. |

**Switching Logic**: All display modes go through the DisplayStack. Only one mode is active at a time. Stopping playback returns to the static background.

### Web Mode Flow (Spotify Example)

1. **Spotify track changes** (librespot webhook → `/spotify/event`)
2. **SpotifyManager** broadcasts via WebSocket:
   ```json
   {
     "event": "track_changed",
     "data": {
       "name": "Track Name",
       "artists": "Artist Name",
       "album": "Album Name",
       "album_art_url": "https://...",
       "duration_ms": 240000
     }
   }
   ```
3. **SpotifyManager** calls `background_manager.start_now_playing_web_mode()`
4. **BackgroundManager** → **ChromiumManager** launches kiosk:
   ```bash
   Xvfb :99 -screen 0 1920x1080x24 &
   DISPLAY=:99 chromium-browser --kiosk http://localhost/now-playing
   ```
5. **Now-Playing Page** connects to WebSocket, receives updates
6. **Display updates** in real-time without regenerating images

### Fallback Mode

If Chromium fails to start (not installed, Xvfb error, etc.):
- System automatically falls back to legacy PIL/FFmpeg video mode
- Error logged but playback continues
- Check logs: `sudo journalctl -u hsg-canvas | grep Chromium`

### Testing Web Features

```bash
# Test WebSocket endpoint (requires wscat or similar)
wscat -c ws://localhost/ws/spotify-events

# Test now-playing page in browser
xdg-open http://localhost/now-playing

# Check active connections
curl http://localhost/ws/status

# Monitor Chromium processes
ps aux | grep chromium
ps aux | grep Xvfb
```

### Future Phases (Not Yet Implemented)

- **Phase 2**: Static background as web page (clock, QR codes, widgets)
- **Phase 3**: Audio visualizations, system dashboards, screensavers

### Troubleshooting

**Chromium not starting**:
```bash
# Check if installed
which chromium-browser xvfb

# Install if missing
sudo apt-get install xvfb chromium-browser

# Check logs
sudo journalctl -u hsg-canvas | grep -i chromium
```

**WebSocket not connecting**:
```bash
# Check endpoint
curl http://localhost/ws/status

# Test from browser console
const ws = new WebSocket('ws://localhost/ws/spotify-events');
ws.onmessage = (e) => console.log(e.data);
```

**Display not showing**:
- Xvfb renders to virtual display `:99` (not physical output)
- For actual HDMI output, may need display routing

## Dependencies

### System Packages

```bash
# Web-based display
sudo apt-get install chromium-browser

# Optional: CEC control
sudo apt-get install cec-utils
```

### Python Packages (venv)

All Python dependencies are in `requirements.txt` and installed in `.venv/`:

```bash
# Install/update dependencies
.venv/bin/pip install -r requirements.txt --break-system-packages
```

**Note**: Raspberry Pi requires `--break-system-packages` flag for pip installs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/0x20) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
