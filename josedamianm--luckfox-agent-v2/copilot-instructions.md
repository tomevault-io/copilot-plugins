## luckfox-agent-v2

> Single source of truth for AI agents working on this project. Read this file completely before making any changes.

# CLAUDE.md — LuckFox Agent V2

Single source of truth for AI agents working on this project. Read this file completely before making any changes.

---

## 1. Project Overview

LuckFox Agent V2 is a voice-activated AI agent running on the **LuckFox Pico Max** (Rockchip RV1106 / ARM Cortex-A7, 256MB DDR3). It uses a dual-process architecture:

- **`luckfox_gui`** (C binary): drives LVGL v9 GUI on a 240×240 ST7789 SPI display, owns GPIO buttons, emits button events over IPC
- **`http_api_server_v2.py`** (Python): HTTP API on port 8080, AI pipeline coordination (STT → LLM → TTS via MacBook), drives C binary state via IPC

```
User presses CTRL button
        ↓
luckfox_gui (C) → IPC event → Python HTTP server
                                    ↓
                          HTTP → MacBook pipeline
                          (audio → STT → LLM → TTS)
                                    ↓
                    IPC command → luckfox_gui (C)
                                    ↓
                        LVGL → SPI → ST7789 240×240
```

### Why Dual-Process?

1. **Performance**: C + LVGL renders at native speed vs Python's ~40ms/frame
2. **LVGL threading**: LVGL is not thread-safe; isolated process avoids sync issues
3. **Stability**: Python crash doesn't kill GUI; display keeps showing last screen
4. **Separation**: GUI rendering vs API/network/audio are fundamentally different workloads

---

## 2. Current Status (2026-03-24)

**Display + all 9 buttons confirmed working on hardware.**

**IPC/HTTP integration complete.** All three Python files speak the V2 protocol:
- `gui_client.py`: `set_state()` sends `{"cmd": "set_state", "state": "...", "text": "..."}` matching `cmd_parser.c`
- `http_api_server_v2.py`: V2 agent state endpoints (`GET/POST /api/agent/state`), CTRL button events drive state machine (pressed→LISTENING, released→THINKING)
- `main.py`: launches `http_api_server_v2.py`

**IDLE screen confirmed working on hardware:**
- Page 0 (Status): time updates every second, date and IP display correctly
- Page 1 (Kawaii face): smooth animation, blink/bounce/emotion state machine working
- LEFT/RIGHT buttons navigate between pages
- Clock uses `time()`/`localtime()`/`strftime()` — pure C stdlib, no shell required

**Camera reworked (2026-03-20):**
- rkipc cycle snapshots **DISABLED** (`enable_cycle_snapshot = 0` in `/oem/usr/share/rkipc-300w.ini`)
- RTSP stream at native **25 FPS**, H.265/HEVC, 2304×1296, with PCM A-law 8kHz audio
- RTSP exposed externally via FRP: `rtsp://luckfoxpico1.aiserver.onmobilespace.com:8554/live/0`
- `/api/capture` uses ffmpeg `select=key` filter → proper IDR keyframe JPEG (~10s response)
- `/api/stream` removed — use RTSP directly in VLC/ffplay
- Static ffmpeg 7.0.2 armhf at `/mnt/sdcard/ffmpeg` (32MB, gitignored, in sync.sh)
- SD card remounted with `fmask=0022` (exec enabled) via udev rule change

**Audio input — INMP441 mic integrated (2026-03-24):**
- INMP441 I2S MEMS microphone wired to ESP32-C3 GPIO10 (I2S DIN)
- Full-duplex I2S: speaker (MAX98357A) and mic (INMP441) share BCLK (GPIO0) / LRC (GPIO1)
- ESP32-C3 firmware: `luckfox_audio_fulldup.ino` — captures 32-bit I2S, converts to 16-bit PCM (`raw32[i] >> 8`), streams upstream via `PKT_MIC_DATA` over UART2
- Python `MicReceiver` class in `audio_sender.py` — background thread reads mic packets, buffers PCM, exports WAV on demand
- Mic lifecycle tied to `AUDIO_START`/`AUDIO_STOP` — mic starts/stops with speaker session
- HTTP endpoints: `/api/audio/record/start`, `/api/audio/record/stop`, `/api/audio/record/download`, `/api/audio/stream`
- Bandwidth: 512 kbps bidirectional audio (mic + speaker) within UART's 736 kbps capacity

**Live stream client fixed (2026-03-24):**
- `luckfox_remote.py stream` uses two separate ffplay processes (video RTSP + audio HTTP)
- Previous pipe-mux approach (ffmpeg → NUT → ffplay) failed due to rkipc's ~15s GOP — P-frames flooded pipe before IDR keyframe
- Direct RTSP → ffplay handles long GOP correctly (same as VLC)

**Pending — next actions:**

1. **MacBook AI pipeline** — receive recorded audio from board, run STT (speech-to-text), send transcript to LLM, get response, run TTS (text-to-speech), send WAV back to board
2. **Board playback** — receive TTS WAV via `/api/audio/play`, play through ESP32-C3, set SPEAKING state with response text, return to IDLE when playback completes
3. **Error handling** — on any pipeline failure (network timeout, STT error, LLM error), set ERROR state with message, then return to IDLE after a few seconds
4. **End-to-end test** — press CTRL, speak, release CTRL → board records → MacBook processes → board speaks response → IDLE

---

## 3. Agent State Machine

Five states. The C binary renders the correct screen for each state. Python drives state transitions via IPC.

| State | Trigger | Visual |
|-------|---------|--------|
| `IDLE` | Boot / pipeline complete | Logo + "Press CTRL to start" |
| `LISTENING` | CTRL pressed | Mic/waveform animation, green |
| `THINKING` | CTRL released | Spinner/dots, orange |
| `SPEAKING` | LLM response ready | Response text + waveform, cyan |
| `ERROR` | Any failure | Error message, red |

```
IDLE ──[CTRL press]──► LISTENING ──[CTRL release]──► THINKING
                                                          │
                                              [response ready]
                                                          ▼
IDLE ◄──[playback done]── SPEAKING ◄──[LLM done]── THINKING
                                                          │
IDLE ◄──────────────── ERROR ◄────────[any failure]──────┘
```

---

## 4. Screen Layouts (240×240, dark background)

### IDLE
- Centered brand text: `"LUCKFOX AGENT"` — Montserrat 32px, white
- Sub-text: `"Press CTRL to start"` — 16px, gray `#888888`
- Background: pure black `#000000`

### LISTENING
- Top label: `"Listening..."` — 24px, green `#00FF80`
- Center: 7 animated vertical waveform bars (ping-pong height animation), green `#00FF80`
- Bottom label: `"Release CTRL to process"` — 14px, gray `#888888`

### THINKING
- Center: spinning arc animation (300° arc rotates), orange `#FF8800`
- Label below spinner: `"Thinking..."` — 24px, orange `#FF8800`

### SPEAKING
- Top label: `"Speaking..."` — 24px, cyan `#00CCFF`
- Center text area: response text (last ~120 chars that fit), 16px, white, word-wrap
- Bottom: 3 animated pulsing dots, cyan `#00CCFF`

### ERROR
- Center: `"!"` symbol — 48px, red `#FF3333`
- Below: error message text — 16px, red `#FF3333`, truncated to fit

---

## 5. IPC Protocol

Newline-delimited JSON over Unix domain socket `/tmp/luckfox_gui.sock`. Transport: `SOCK_STREAM`, bidirectional. Python client auto-reconnects with 500ms backoff. Up to 4 simultaneous clients.

### Python → C (commands)

```json
{"cmd": "set_state", "state": "idle"}
{"cmd": "set_state", "state": "listening"}
{"cmd": "set_state", "state": "thinking"}
{"cmd": "set_state", "state": "speaking", "text": "response text here"}
{"cmd": "set_state", "state": "error", "text": "error message"}
```

### C → Python (events)

```json
{"event": "button", "name": "CTRL", "state": "pressed"}
{"event": "button", "name": "CTRL", "state": "released"}
```

---

## 6. C Binary Structure (`lvgl_gui/src/`)

| Module | Role |
|--------|------|
| `main.c` | LVGL init, main loop, GPIO button polling, signal handling |
| `hal/disp_driver.c` | ST7789 SPI driver — init, flush_cb (byte swap via `write()` syscall) |
| `hal/disp_driver.h` | Public: `disp_driver_init()`, `disp_driver_deinit()`, `disp_fill_color()` |
| `ipc/ipc_server.c` | Unix socket server, non-blocking, up to 4 clients, `ipc_server_broadcast()` |
| `ipc/ipc_server.h` | Public: `ipc_server_init()`, `ipc_server_poll()`, `ipc_broadcast()` |
| `ipc/cmd_parser.c` | Parse `set_state` JSON → call `agent_set_state()` |
| `ipc/cmd_parser.h` | Public: `cmd_parse()` |
| `screens/scr_agent.c` | All 5 agent states in one file — tick-driven animations |
| `screens/scr_agent.h` | Public: `agent_screen_init()`, `agent_set_state()`, `agent_tick()` |
| `faces/kawaii_face.c` | Animated emoji face — 9 emotions, auto-blink, expression transitions |
| `faces/kawaii_face.h` | Public: `kawaii_init()`, `kawaii_tick()`, `kawaii_set_emotion()` |

### `main.c` Main Loop

```c
lv_init();
disp_driver_init();
agent_screen_init();       // creates LVGL screen, sets IDLE state
ipc_server_init();         // binds /tmp/luckfox_gui.sock
gpio_buttons_init();       // export + set direction, init prev[] from reads

while (running) {
    ipc_server_poll();     // accept new clients, read commands, dispatch
    gpio_poll_buttons();   // detect CTRL press/release → ipc_broadcast() event
    agent_tick();          // drive animations (waveform bars, spinner, dots)
    lv_timer_handler();    // LVGL render cycle
    usleep(10000);         // 10ms = 100Hz loop
}
```

### `scr_agent.c` Structure

Single LVGL screen with overlapping containers, one shown per state:

```c
typedef enum { STATE_IDLE, STATE_LISTENING, STATE_THINKING, STATE_SPEAKING, STATE_ERROR } agent_state_t;

void agent_screen_init(void);           // create all 5 state containers on one lv_screen
void agent_set_state(agent_state_t s, const char *text);  // hide all, show target, start anim
void agent_tick(void);                  // called every loop iteration — advance animations
```

Animation approach (no LVGL anim timers, driven by `agent_tick()`):
- **LISTENING bars**: 7 `lv_obj` rectangles, heights oscillate with per-bar phase offset
- **THINKING spinner**: single `lv_arc`, start angle incremented by 4° per tick
- **SPEAKING dots**: 3 circles, opacity pulses with phase offset per dot

---

## 7. Python-Side Files

| File | Role |
|------|------|
| `board/sdcard/gui_client.py` | IPC client class — connects to `/tmp/luckfox_gui.sock`, auto-reconnects, sends JSON, reads button events in background thread |
| `board/sdcard/http_api_server_v2.py` | HTTP API on port 8080 — agent state, camera capture/stream, audio playback via ESP32-C3; uses `ThreadingHTTPServer` (per-request thread) so MJPEG stream and other API calls run concurrently |
| `board/root/main.py` | Boot launcher — starts `luckfox_gui` binary, waits 1.5s, then starts Python server |
| `board/sdcard/audio_sender.py` | UART packet protocol to ESP32-C3 — includes `AudioSender` (playback) and `MicReceiver` (mic capture/streaming) |
| `client/luckfox_remote.py` | MacBook CLI client for testing all API endpoints — `stream` command opens RTSP video + HTTP mic audio |

---

## 8. Hardware Configuration (Confirmed Working)

### Display — ST7789 240×240

| Item | Value |
|------|-------|
| SPI device | `/dev/spidev0.0` @ 32MHz, `SPI_MODE_0` |
| SPI transfer | **`write()` syscall** — `ioctl(SPI_IOC_MESSAGE)` silently fails on RV1106 |
| DC GPIO | 73 |
| RST GPIO | 51 |
| BL GPIO | 72 |
| MADCTL | `0x60` (90° landscape) |
| Window offset | XOFF=80, YOFF=0 |
| Color format | RGB565 big-endian — manual byte swap in `flush_cb` |
| LVGL render mode | `LV_DISPLAY_RENDER_MODE_FULL`, single 240×240 buffer, `NULL` second buffer |
| LVGL refresh | `lv_refr_now(NULL)` for immediate flush; `lv_timer_handler()` in main loop |

### Buttons — 9 GPIO, active-low (idle=1, pressed=0)

| Button | GPIO |
|--------|------|
| A | 57 |
| B | 69 |
| X | 65 |
| Y | 67 |
| UP | 55 |
| DOWN | 64 |
| LEFT | 68 |
| RIGHT | 66 |
| CTRL | 54 |

- GPIOs must be **exported** and set to `in` direction by the binary on every startup (not persistent across reboots)
- Use **direct sysfs polling** in main loop (LVGL v9 keypad indev `read_cb` not polled without focused group)
- Initialize `prev[]` state from actual GPIO reads to prevent false triggers on startup
- Only CTRL button events are emitted over IPC (others reserved for future use)
- Hardware pull-ups configured via IOC registers (`board/init.d/S99button_pullups`)

### Audio — ESP32-C3 Full-Duplex via UART2

| Item | Value |
|------|-------|
| Device | `/dev/ttyS2` |
| Baud | 921600 |
| Protocol | Binary: SYNC `0xAA55`, packet type, length, payload, XOR checksum |
| LuckFox TX | GPIO42 (Pin1) → ESP32-C3 GPIO4 |
| LuckFox RX | GPIO43 (Pin2) ← ESP32-C3 GPIO7 |
| Audio input | INMP441 I2S mic → ESP32-C3 GPIO10 (DIN) → `PKT_MIC_DATA` → `/dev/ttyS2` |
| Audio output | `/dev/ttyS2` → `PKT_AUDIO_DATA` → ESP32-C3 GPIO2 (DOUT) → MAX98357A speaker |
| I2S shared bus | BCLK=GPIO0, LRC/WS=GPIO1 (both mic and speaker share clock) |
| Mic format | 16kHz, 16-bit mono (INMP441 outputs 24-bit in 32-bit I2S slot, firmware converts via `raw32[i] >> 8`) |
| Amp enable | GPIO3 (SD/EN pin on MAX98357A) — HIGH = enabled |
| Firmware | `luckfox_audio_fulldup.ino` (full-duplex, replaces speaker-only `luckfox_audio_receiver.ino`) |
| Note | RV1106 SoC has audio ADC but LuckFox Pico Max has **no onboard mic** — all audio goes through ESP32-C3 |

---

## 9. LVGL v9 Critical Notes

These are hard-won lessons. Do NOT change these without testing on hardware:

- `lv_timer_handler()` **blocks** in partial render mode → must use `LV_DISPLAY_RENDER_MODE_FULL`
- `LV_COLOR_16_SWAP 1` is a v8 macro, **silently ignored** in v9 → manual byte swap in `flush_cb`
- `LV_COLOR_FORMAT_RGB565_SWAP` enum **does not exist** in this build → use `LV_COLOR_FORMAT_RGB565`
- LVGL v9 keypad indev requires a focused group for `read_cb` to be polled → bypass with direct GPIO loop
- Call `lv_refr_now(NULL)` after style/content changes to force immediate screen refresh
- Prefer tick-driven animations in `agent_tick()` over LVGL anim timers for predictability
- **`popen("date ...")`  silently fails on RV1106** (no shell at that path) → always use `time()`/`localtime()`/`strftime()` for system time
- **`lv_timer_ready()`** may not exist in all LVGL v9 builds → call the callback directly instead
- **Clock update pattern (confirmed working)**: call `update_clock()` inside `agent_tick()` every tick; use a static `time_t g_last_second` to detect second changes; call `lv_refr_now(NULL)` every tick (same as face animation page)

### LVGL Configuration (`lvgl_gui/lv_conf.h`)

- `LV_COLOR_DEPTH 16` (RGB565)
- `LV_COLOR_16_SWAP 0` (ignored in v9; byte swap done manually)
- Buffer: 48KB, DPI: 200, display: 240×240
- Fonts enabled: Montserrat 12, 16, 24, 32, 48
- Tick: custom via `gettimeofday()`
- File system: POSIX, letter `S`, path `/mnt/sdcard`

---

## 10. Build System

### Cross-Compilation

Uses Docker container `luckfox-crossdev:1.0` with Rockchip toolchain at `/toolchain`.

**Toolchain**: `arm-rockchip830-linux-uclibcgnueabihf-gcc`
**Flags**: `-march=armv7-a -mfpu=neon-vfpv4 -mfloat-abi=hard -O2`
**Auto-detection order**: `TOOLCHAIN_PATH` env → `LUCKFOX_SDK` env → `/toolchain`

#### Start Docker Container

```bash
docker run -it --name luckfox-crossdev \
  -v ~/luckfox-crossdev/projects:/workspace \
  luckfox-crossdev:1.0 /bin/bash

# If already exists:
docker start -i luckfox-crossdev
```

#### LVGL Submodule Setup (first time only)

```bash
git submodule add https://github.com/lvgl/lvgl.git lvgl_gui/lib/lvgl
cd lvgl_gui/lib/lvgl && git checkout release/v9.2 && cd ../../..

# Or if .gitmodules already exists:
git submodule update --init --recursive
cd lvgl_gui/lib/lvgl && git checkout release/v9.2 && cd ../../..
```

#### Build

```bash
cd lvgl_gui
mkdir -p build && cd build
cmake .. -DCMAKE_TOOLCHAIN_FILE=../toolchain-rv1106.cmake -DCMAKE_BUILD_TYPE=Release
make -j$(nproc)
# Output: lvgl_gui/build/luckfox_gui (ELF 32-bit ARM)
```

#### Verify Binary

```bash
file lvgl_gui/build/luckfox_gui
# Expected: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV)
```

#### Rebuild After Changes

The Docker container runs on a remote Ubuntu server (`nlighten_gpu_server`). The workspace is mounted at `/home/josemanco/luckfox-crossdev/projects/` on the server and at `/workspace/` inside the container. Follow these steps end-to-end whenever the C binary (`luckfox_gui`) needs to be recompiled.

##### Step 1 — Push changes from MacBook to GitHub

```bash
git add -A && git commit -m "your message"
git push
```

##### Step 2 — Start container on remote server and open a shell

```bash
ssh nlighten_gpu_server
docker start luckfox-crossdev && docker exec -it luckfox-crossdev /bin/bash
```

##### Step 3 — Pull latest code inside container

```bash
cd /workspace/LuckFox_Agent_v2
git pull
```

##### Step 4 — Compile (clean build)

```bash
cd lvgl_gui
rm -rf build && mkdir build && cd build
cmake -DCMAKE_TOOLCHAIN_FILE=../toolchain-rv1106.cmake .. && make -j$(nproc)
ls -ltrh   # verify luckfox_gui binary was produced
```

##### Step 5 — Copy binary back to MacBook

```bash
# Run on MacBook (not inside container/server)
cd /path/to/LuckFox_Agent_v2
scp nlighten_gpu_server:/home/josemanco/luckfox-crossdev/projects/LuckFox_Agent_v2/lvgl_gui/build/luckfox_gui board/executables/luckfox_gui
ls -ltrh board/executables/luckfox_gui   # verify
```

##### Step 6 — Push binary to board via sync.sh

```bash
./sync.sh push
```

##### Step 7 — Reboot board to apply

```bash
ssh root@192.168.1.60 "reboot"
# or via tunnel:
# ssh root@luckfoxpico1.aiserver.onmobilespace.com "reboot"
```

---

## 11. Deployment

### Sync Script (preferred)

```bash
./sync.sh              # rsync push to board (remote via SSH tunnel)
./sync.sh --private    # push via local network (192.168.1.60)
./sync.sh pull         # pull from board
./sync.sh diff         # show differences
./sync.sh status       # show sync status
```

Remote: `root@luckfoxpico1.aiserver.onmobilespace.com:8022`
Local: `root@192.168.1.60`

### Manual Deploy

```bash
scp lvgl_gui/build/luckfox_gui root@192.168.1.60:/root/Executables/luckfox_gui
ssh root@192.168.1.60 "killall luckfox_gui; /root/Executables/luckfox_gui"
```

### Board File Locations

| Item | Path |
|------|------|
| C binary | `/root/Executables/luckfox_gui` |
| Python scripts | `/mnt/sdcard/` |
| ffmpeg binary | `/mnt/sdcard/ffmpeg` (32MB, static armhf) |
| IPC socket | `/tmp/luckfox_gui.sock` |
| Autostart | `/etc/init.d/S99luckfox_agent` |
| Emoji PNGs | `/mnt/sdcard/emoji/<name>.png` |
| HTTP API port | `8080` |
| RTSP stream | `rtsp://luckfoxpico1.aiserver.onmobilespace.com:8554/live/0` |

### Boot Sequence

```
S20spi0overlay          ← Enable /dev/spidev0.0
S21uart2overlay         ← Enable UART2 for audio
S50rtcinit              ← RTC initialization
S60ntpd                 ← NTP time sync
S98frpc                 ← FRP tunnel
S99button_pullups       ← GPIO pull-up registers
S99luckfox_agent        ← Launch luckfox_gui, then Python server
```

### Autostart Script

```sh
#!/bin/sh
start() {
    /root/Executables/luckfox_gui &
    sleep 2
    python3 /mnt/sdcard/http_api_server_v2.py &
}
stop() { pkill luckfox_gui; pkill -f http_api_server_v2.py; }
case "$1" in
    start) start ;; stop) stop ;; restart) stop; sleep 1; start ;;
esac
```

---

## 12. Camera — rkipc Integration

The camera is managed entirely by the **rkipc** service (Rockchip IPC daemon). Do NOT run a custom camera_daemon — it would conflict with rkipc and lose ISP/3A processing, producing dark/greenish images.

### How rkipc works

- Started automatically at boot via `RkLunch.sh` → `post_chk()` → `rkipc -a /oem/usr/share/iqfiles`
- Reads config from `/userdata/rkipc.ini` (copied from `/oem/usr/share/rkipc-300w.ini` on every boot)
- Provides RTSP stream at `rtsp://<device-ip>/live/0` (main, 2304×1296) and `/live/1` (sub, 704×576)
- Both streams: H.265/HEVC, 25 FPS, with PCM A-law 8kHz mono audio
- Cycle snapshots **disabled** — RTSP is the only output
- **GOP (keyframe interval) is ~15 seconds** — only 1 IDR frame per ~375 frames; clients joining mid-stream see HEVC RPS errors until next IDR
- Direct RTSP clients (VLC, `ffplay -rtsp_transport tcp`) handle long GOP gracefully; pipe-mux approaches (ffmpeg → NUT pipe → ffplay) do not
- IPC socket: `/var/tmp/rkipc` (Unix domain socket) — binary protocol, not JSON

### Key config (persisted in `/oem/usr/share/rkipc-300w.ini`)

**IMPORTANT**: Always edit `/oem/usr/share/rkipc-300w.ini` for persistent changes. `/userdata/rkipc.ini` is overwritten on every boot by `RkLunch.sh`.

```ini
[video.source]
enable_jpeg = 1
enable_rtsp = 1

[video.jpeg]
enable_cycle_snapshot = 0       ; DISABLED — RTSP provides full 25 FPS

[storage]
mount_path = /mnt/sdcard        ; NOT /userdata (only 2.2MB, always full)
```

### Python API (`http_api_server_v2.py`)

| Constant / Function | Behaviour |
|---|---|
| `RTSP_URL = "rtsp://127.0.0.1/live/0"` | Local RTSP stream from rkipc |
| `FFMPEG = "/mnt/sdcard/ffmpeg"` | Static armhf ffmpeg binary |
| `rkipc_running()` | Returns `True` if `/var/tmp/rkipc` exists as a socket |
| `capture_frame()` | Runs ffmpeg `select=key` → temp file → reads and returns JPEG bytes |
| `GET /api/capture` | Returns keyframe JPEG (~10s, software H.265 decode) |
| `GET /api/camera/status` | Returns `rkipc_running`, `rtsp_url` |

### Static ffmpeg binary

- Location on board: `/mnt/sdcard/ffmpeg` (SD card, exec-enabled after fmask fix)
- Local repo: `board/executables/ffmpeg` (gitignored, tracked in `sync.sh` FILE_MAP)
- Version: 7.0.2 static armhf from johnvansickle.com (32MB)
- **No Rockchip MPP hardware decode** — software H.265 only, slow on RV1106
- Download on board: `python3 -c "import urllib.request; urllib.request.urlretrieve('https://johnvansickle.com/ffmpeg/releases/ffmpeg-release-armhf-static.tar.xz', '/tmp/ffmpeg.tar.xz')"`

### ffmpeg capture pattern (critical)

```python
# CORRECT — write to temp file, discard stdout/stderr
subprocess.run(
    [FFMPEG, '-rtsp_transport', 'tcp', '-i', RTSP_URL,
     '-vf', 'select=key', '-frames:v', '1', '-f', 'image2', '-update', '1', '/tmp/capture.jpg', '-y'],
    stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL, timeout=20
)
# WRONG — capture_output=True causes pipe deadlock (ffmpeg floods stderr)
# WRONG — pipe:1 with image2 format produces grey/corrupt frames
```

### SD card exec fix

- `/lib/udev/rules.d/61-sd-cards-auto-mount.rules`: changed `fmask=133` → `fmask=022`
- Allows executing binaries from `/mnt/sdcard/`
- Persistent across reboots. Requires reboot to apply (exFAT ignores remount for fmask).

### FRP — RTSP tunnel

- `frpc.toml`: `name=luckfox_rtsp`, `localPort=554`, `remotePort=8554`
- FRP server `docker-compose.yaml` must explicitly expose `"8554:8554"` (unlike HTTP ports proxied internally by Caddy)
- Client must force TCP: `vlc` or `ffplay -rtsp_transport tcp rtsp://luckfoxpico1.aiserver.onmobilespace.com:8554/live/0`

### RTSP stream (VLC / external)

```
rtsp://<device-ip>/live/0   ← main stream (2304×1296)
rtsp://<device-ip>/live/1   ← sub stream
```

---

## 13. V1 Reference (Legacy)

Key V1 files still in the repo (deprecated):

| File | Description |
|------|-------------|
| `board/sdcard/http_api_server.py` | V1 monolith (replaced by `http_api_server_v2.py`) |
| `board/init.d/S99python` | V1 boot script (starts `main.py` which launches the old server) |

---

## 14. Project Directory Structure

```
LuckFox_Agent_v2/
├── CLAUDE.md                        ← This file (single source of truth)
├── sync.sh                          ← rsync deploy script
│
├── board/                           ← Files deployed to the board
│   ├── root/
│   │   ├── main.py                  ← Boot launcher (starts luckfox_gui + Python server)
│   │   ├── enable_spi0_spidev.dtbo
│   │   ├── enable_spi0_spidev.dts
│   │   └── enable_uart2.dts
│   ├── sdcard/
│   │   ├── http_api_server_v2.py    ← V2 HTTP API server (IPC-based)
│   │   ├── gui_client.py            ← Python IPC client class
│   │   ├── audio_sender.py          ← UART audio to ESP32-C3
│   │   └── http_api_server.py       ← V1 legacy (deprecated)
│   ├── executables/
│   │   └── luckfox_gui              ← Compiled ARM binary
│   └── init.d/
│       ├── S20spi0overlay
│       ├── S21uart2overlay
│       ├── S50rtcinit
│       ├── S60ntpd
│       ├── S98frpc
│       ├── S99button_pullups
│       └── S99python
│
├── lvgl_gui/                        ← LVGL C application
│   ├── CMakeLists.txt
│   ├── toolchain-rv1106.cmake
│   ├── lv_conf.h
│   ├── src/
│   │   ├── main.c
│   │   ├── hal/
│   │   │   ├── disp_driver.c       ← ST7789 SPI driver (write() syscall, manual byte swap)
│   │   │   └── disp_driver.h
│   │   ├── ipc/
│   │   │   ├── ipc_server.c        ← Unix socket server, non-blocking, 4 clients
│   │   │   ├── ipc_server.h
│   │   │   ├── cmd_parser.c        ← JSON command parser (currently only set_state)
│   │   │   └── cmd_parser.h
│   │   ├── screens/
│   │   │   ├── scr_agent.c          ← All 5 agent states, tick-driven animations
│   │   │   └── scr_agent.h
│   │   └── faces/
│   │       ├── kawaii_face.c        ← Animated emoji face (9 emotions, blink, bounce)
│   │       └── kawaii_face.h
│   └── lib/
│       └── lvgl/                    ← LVGL v9.2 (git submodule)
│
├── client/                          ← Desktop client tools
│   └── luckfox_remote.py            ← MacBook CLI (status, capture, stream, audio, etc.)
└── luckfox_audio_receiver/          ← ESP32-C3 firmware (full-duplex: speaker + INMP441 mic)
    └── luckfox_audio_fulldup.ino    ← Active firmware (replaces luckfox_audio_receiver.ino)
```

---

## 15. Troubleshooting

| Symptom | Fix |
|---------|-----|
| Display stays blank/red after boot | SPI not working — check `/dev/spidev0.0` exists; must use `write()` not `ioctl` |
| Blue flash then black screen | LVGL render issue — ensure `lv_init()` before `disp_driver_init()` |
| Buttons not responding | Check GPIO export: `ls /sys/class/gpio/gpio57`; direction must be `in` |
| False button trigger on startup | Initialize `prev[]` from actual GPIO reads before loop |
| Text/image mirrored | Check MADCTL=`0x60`, XOFF=80, YOFF=0 |
| IPC socket not found | `luckfox_gui` failed — run in foreground, check stderr |
| `arm-rockchip830...: not found` | `export TOOLCHAIN_PATH=/toolchain/bin/arm-rockchip830-linux-uclibcgnueabihf-` |
| IPC connection refused (Python → C) | `luckfox_gui` must start first; socket created after LVGL init |
| `git submodule` fails in container | Check internet: `curl -I https://github.com`; or clone LVGL on host and copy in |
| `python3` not found on board | `opkg update && opkg install python3` |
| `/api/capture` returns error "rkipc not running" | Check `ps aux | grep rkipc`; if missing, `RkLunch.sh` may have failed — reboot |
| `/api/capture` takes >20s or times out | Software H.265 decode is slow — `select=key` needs ~10s; verify ffmpeg at `/mnt/sdcard/ffmpeg` |
| rkipc config changes not persisting across reboot | Edit `/oem/usr/share/rkipc-300w.ini` — NOT `/userdata/rkipc.ini` (overwritten on boot) |
| `pkill` not found on board | Use `kill $(ps \| grep <name> \| grep -v grep \| awk '{print $1}')` |
| BusyBox `wget` rejects HTTPS URLs | Use Python: `python3 -c "import urllib.request; urllib.request.urlretrieve(url, path)"` |
| `curl` not found on board | Same — use Python urllib |
| ffmpeg capture returns grey/corrupt JPEG | Don't use `pipe:1` with `-f image2` — write to a temp file instead |
| ffmpeg subprocess hangs/deadlocks in Python | Never use `capture_output=True` — use `stdout=DEVNULL, stderr=DEVNULL` |
| ffmpeg capture times out at 10s | `select=key` takes ~7-10s on RV1106 software decode — use timeout=20 |
| `/api/stream` returns 404 | Endpoint removed — use RTSP: `rtsp://luckfoxpico1.aiserver.onmobilespace.com:8554/live/0` |
| New FRP proxy not reachable externally | Must add port to BOTH `frpc.toml` (board) AND FRP server `docker-compose.yaml` ports section |
| Binary on SD card won't execute (Permission denied) | SD card was `fmask=133` — fixed to `fmask=022` in `/lib/udev/rules.d/61-sd-cards-auto-mount.rules`, needs reboot |
| `stream` command shows noise/dark image | rkipc uses ~15s GOP; pipe-mux fails before first IDR. Fixed: uses two separate ffplay processes (RTSP video + HTTP audio) |
| `stream` video black with `stderr=subprocess.PIPE` | Never use `stderr=subprocess.PIPE` with ffmpeg without reading it — pipe buffer fills up (~64KB) and deadlocks. Use `stderr=subprocess.DEVNULL` |
| Mic audio is silent / flat line | INMP441 L/R pin must be tied to GND (selects LEFT channel) |
| Mic audio distorted / clipped | Verify 32→16 bit conversion uses `raw32[i] >> 8` not `>> 16` |
| No `PKT_MIC_DATA` received from ESP32 | Check GPIO10 ↔ INMP441 SD wire; verify `luckfox_audio_fulldup.ino` is flashed (not old receiver-only firmware) |

### Debug Commands

```bash
# Board connectivity
ssh root@192.168.1.60 "ps aux | grep -E 'luckfox_gui|http_api|rkipc'"
dmesg | tail -30

# GUI / IPC
ssh root@192.168.1.60 "killall luckfox_gui; /root/Executables/luckfox_gui"
ls -la /tmp/luckfox_gui.sock

# Camera / RTSP
ffprobe -v quiet -print_format json -show_streams -rtsp_transport tcp rtsp://luckfoxpico1.aiserver.onmobilespace.com:8554/live/0
# Check rkipc keyframe interval (should show 1 key_frame=1 per ~15s)
ffprobe -rtsp_transport tcp -select_streams v:0 -show_entries frame=key_frame,pts_time -read_intervals "%+15" rtsp://luckfoxpico1.aiserver.onmobilespace.com:8554/live/0 2>/dev/null | grep key_frame=1
ssh root@192.168.1.60 "df -h /mnt/sdcard"
ssh root@192.168.1.60 "grep 'enable_cycle_snapshot' /oem/usr/share/rkipc-300w.ini"

# Client testing (from MacBook)
python3 client/luckfox_remote.py camera-status
python3 client/luckfox_remote.py capture -o /tmp/frame.jpg && open /tmp/frame.jpg  # ~10s
python3 client/luckfox_remote.py stream  # prints RTSP URLs
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/josedamianm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
