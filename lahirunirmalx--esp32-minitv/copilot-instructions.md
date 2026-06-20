## esp32-minitv

> Firmware project for the **Freenove ESP32 Mini TV (FNK0112)** — a tiny

# CLAUDE.md

Firmware project for the **Freenove ESP32 Mini TV (FNK0112)** — a tiny
TV-shaped desk gadget. Pure ESP-IDF v5.5 (no Arduino), fully standalone:
LVGL 9.2 and Mooncake 2.1 are vendored in `components/` and the build works
offline on a fresh checkout.

## The device in one paragraph

Classic ESP32 ("ESP-32S" module), 4 MB flash, **no PSRAM**. 240×240 ST7789
IPS over 80 MHz SPI. **The display is not a touchscreen** — the only input
is one capacitive pad on top of the cube (tap = next app, 3 s hold = setup
portal). WiFi STA + BLE (NimBLE, advertises as "MiniTV") + HTTPS web client.
The UI theme follows the device's shell color (black/white/orange, or a
custom palette), chosen in the captive portal and stored in NVS. The full
pin map lives in [main/hal/hal_config.h](main/hal/hal_config.h) — every pin
the firmware uses is defined there.

## Skills (use them — they encode the device's rules)

- **minitv-create-app** (`.claude/skills/minitv-create-app/`) — any request
  to create/add an app, screen, animation, or display feature. Drives
  `scripts/new_app.py`, enforces the theme layer, gesture-only input, and
  the core-1-UI / core-0-work split.
- **minitv-build-flash** (`.claude/skills/minitv-build-flash/`) — any
  build/flash/monitor request, and the final step after app changes.

## Commands

```bash
./scripts/flash.sh                      # Linux/macOS: build + flash + monitor (port auto-detect)
./scripts/flash.sh build                # build only
./scripts/setup_idf.sh                  # one-time ESP-IDF v5.5 install (flash.sh offers it too)
python scripts/new_app.py <name>        # scaffold + register a new app
python scripts/new_app.py <name> --remove
# Windows: .\scripts\flash.ps1 (IDF via the official installer, v5.5)
# manual: source ~/esp/esp-idf-v5.5/export.sh && idf.py build|flash|monitor
```

## Layout

```text
main/
├── app_main.cpp        # supervisor: ui_task (core 1, LVGL) + net_task (core 0)
├── hal/                # hal_config.h (PINS), display/ (st7789 + lvgl_port),
│                       # touch/ (top pad gestures), backlight/
├── ui/                 # theme.{h,cpp} (shell palettes), screen_app.h (app base)
├── net/                # wifi_manager, captive_portal, web_client, ble_service
└── apps/               # apps.cpp (registry w/ <<APP_*>> markers), app_demo/
components/             # vendored lvgl + mooncake — see components/VENDORED.md
```

## Code explanations (audience setting)

```text
Audience: beginner
```

After writing or changing firmware code, the skills explain it according to
this setting:

- **beginner** (default) — after the build/flash step, walk through every
  added/changed piece in simple terms: **where** it went (file + section),
  **why** it's needed, and **what it does** when the device runs. Follow the
  template in `.claude/skills/minitv-create-app/references/explaining-code.md`.
- **expert** — skip explanations entirely; just report what changed and the
  build/flash result.

If the user says anything like "I'm experienced", "skip the explanations",
"no need to explain", update the line above to `Audience: expert` (and back
to `beginner` on request, e.g. "explain things to me"). A one-off override in
either direction ("explain this one" / "skip the lecture") does not change
the setting.

## Hard rules

- **Never block the UI task.** LVGL calls only from `build()`/`tick()`/
  `teardown()` (core 1). HTTP/JSON/slow work goes on core 0 (`std::thread`
  lands there via sdkconfig) and reports back via atomics/queues.
- **Never hardcode colors** in apps — `theme::palette()` only.
- **ASCII-only punctuation in all generated text** (code, strings, comments,
  portal HTML): never `—`, `·`, `…`, or curly quotes — always plain `-`, `|`,
  `...`, `'`. In display strings this is a hard failure: the built-in
  Montserrat fonts cover ASCII plus only `°` and `•`; anything else renders
  as a blank gap on the device.
- **No positional-touch UI** (sliders/buttons/swipes) — gesture-only input.
- Don't re-enable `CONFIG_ESP_BROWNOUT_DET` and don't remove the IRAM-relief
  block in `sdkconfig.defaults` — both break this board (see comments there).
- **Worker threads: one immortal worker per app, spawned on first open and
  gated by an atomic — never a thread per open.** Rapid app cycling leaks
  per-open workers; a failed thread creation abort()s the firmware
  (exceptions disabled). See app_weather for the pattern.
- New apps go through `scripts/new_app.py`, never hand-wired.
- `sdkconfig.defaults` changes need `rm sdkconfig` + rebuild to apply.

---
> Source: [lahirunirmalx/esp32-minitv](https://github.com/lahirunirmalx/esp32-minitv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
