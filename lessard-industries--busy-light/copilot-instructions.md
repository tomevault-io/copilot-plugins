## busy-light

> Workplace availability indicator: 5 deployed ESP32 devices poll an M365 calendar (via per-person Google Apps Script ICS parsers) and show meeting status on LEDs; state aggregates over HiveMQ MQTT to a Node-RED dashboard on lessard-cloud (moved from the Oracle micro 2026-07-31), plus an optional Pi Zero e-ink display.

# Busy Light

Workplace availability indicator: 5 deployed ESP32 devices poll an M365 calendar (via per-person Google Apps Script ICS parsers) and show meeting status on LEDs; state aggregates over HiveMQ MQTT to a Node-RED dashboard on lessard-cloud (moved from the Oracle micro 2026-07-31), plus an optional Pi Zero e-ink display.

**Full architecture, MQTT topic map, schedule, and to-dos live in the Project Status MCP under "Busy Light" — read it at session start.**

## Components in this repo

| Path | What | Deploys to |
|---|---|---|
| `src/main.cpp` | ESP32 firmware (one build, per-device GPIO via conditional compile) | 5 ESP32 devices, USB `COM3` |
| `Google_Script/Code.js` | Apps Script calendar parser (canonical copy) | **5 separate Apps Script web-app deployments** — see below |
| `Node-RED/flow.json` | Dashboard flow | Node-RED on lessard-cloud (busylight.lessardindustries.com) |
| `Raspberry_Pi/busy_light_status.py` | E-ink display driver | Pi Zero W |

## The 5-deployment gotcha (IMPORTANT)

There is **one Apps Script project per device**, each pointing at a different person's ICS feed (URL in that project's Script Properties, not in the repo). Any `Code.js` fix must be pasted into **all 5 projects individually** and each **redeployed** (Deploy → Manage deployments → New version) — saving alone doesn't update the `/exec` URL. The 5 live copies have **drifted** (e.g. `parseICSDate(line)` vs `parseICSDateString(dateStr)` — two different functions, both must exist; match existing call sites when pasting). Verify a deploy via the endpoint JSON: `debug_info.script_version`.

## Build / flash

- Single PlatformIO env `esp32doit-devkit-v1`, upload + monitor on `COM3`, 115200.
- **Platform pinned to espressif32@6.9.0 (ESP32 Arduino 2.0.17) — do NOT upgrade to 3.x, it breaks LED PWM.** Same pin policy as the other LED projects.
- Flashing is per-device over USB; Ed physically plugs each one in — coordinate with him, don't assume a device is connected.

## Versions (bump both when releasing)

Firmware `v8.x` in `src/main.cpp`; Apps Script version constant in `Code.js` (surfaced via `debug_info.script_version`).

---
> Source: [Lessard-Industries/Busy-Light](https://github.com/Lessard-Industries/Busy-Light) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
