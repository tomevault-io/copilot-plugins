## xiao-ble-motion

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SeeedUA Smart Retail — an IoT system that detects when products are picked up from shelves using BLE motion sensors and triggers video playback on screens via MQTT. Three components:

1. **nRF52840 Sensor Firmware** (`XIAO_nRF52840_LowPowerMotionDetect/`) — Arduino C++, runs on Seeed XIAO nRF52840 Sense with LSM6DS3 IMU. Sleeps at <5µA, wakes on motion, broadcasts BTHome v2 BLE packets, returns to sleep.
2. **ESP32 Gateway Firmware** (`XIAO_ESP32_Series_BluetoothProxy/`) — Arduino C++, runs on XIAO ESP32-C3/S3/C6/C5. Scans BLE, converts BTHome broadcasts to JSON, publishes over MQTT.
3. **FastAPI Backend** (`app/backend/`) — Python. Manages product-to-video mappings, consumes MQTT sensor events, deduplicates, publishes screen playback commands. Serves admin web UI.

## Build & Development

### Arduino Firmware (both projects)

Developed in Arduino IDE. No Makefile or PlatformIO — open the `.ino` file directly.

**nRF52840 Sensor:**
- Board package: Seeed nRF52 Boards (Adafruit BSP)
- Library: Seeed Arduino LSM6DS3
- Board selection: Seeed XIAO nRF52840 Sense
- FQBN: `Seeeduino:nrf52:xiaonRF52840Sense`
- CLI compile: `arduino-cli compile --fqbn Seeeduino:nrf52:xiaonRF52840Sense XIAO_nRF52840_LowPowerMotionDetect/`
- Serial monitor: 115200 baud
- USB mode AT commands for runtime config: `AT+HELP`, `AT+INFO`, `AT+THRESHOLD=0x0A`, `AT+TAILWINDOW=3000`, `AT+TXPOWER=4`, `AT+SAVE`, `AT+DEFAULT`, `AT+REBOOT`

**ESP32 Gateway:**
- Board package: Espressif ESP32 Arduino
- Libraries: WiFiManager, PubSubClient, ArduinoJson, NimBLE
- Partition scheme: Minimal SPIFFS
- Supports C3/S3/C6/C5 variants (pin mappings auto-detected via board defines)

### Python Backend

```bash
cd app/backend
uv sync                           # install dependencies
uv run python main.py             # http://localhost:8080
# or
uv run uvicorn main:app --reload --host 0.0.0.0 --port 8080
```

Configuration via environment variables or `.env` file (see `config.py` — Pydantic BaseSettings). Uses `uv` for dependency management (`pyproject.toml` + `uv.lock`).

## Architecture — Data Flow

```
nRF52840 Sensor (BLE broadcast, BTHome UUID 0xFCD2)
    → ESP32 Gateway (BLE scan → MQTT publish to bthome/{mac}/state)
    → MQTT Broker (Mosquitto)
    → FastAPI Backend (dedup 2s window → lookup product_map.csv → publish screen/{id}/play)
    → Display Player (plays video)
```

## nRF52840 Sensor — Modular Architecture

Thin `.ino` entry point → `app_main.h/cpp` state machine orchestrator → module layer → shared types. Files use prefix-based naming by domain: `core_*` (types/ISR/debug), `bsp_*` (board support), `sensor_*` (IMU driver), `comm_*` (BLE/BTHome), `app_*` (application logic).

| File | Prefix/Layer | Purpose |
|------|--------------|---------|
| `XIAO_nRF52840_LowPowerMotionDetect.ino` | Entry | 26-line entry: `static AppContext ctx; setup→appSetup; loop→appLoop` |
| `config.h` | Config (L0) | Deployment config (thresholds, timing, TX power) — **FROZEN, never modify field order** |
| `core_debug.h` | `core_` (L0) | Debug print macros (header-only) |
| `bsp_pins.h` | `bsp_` (L0) | Hardware pin definitions (header-only) |
| `core_types.h` | `core_` (L1) | Shared types: `RuntimeConfig`, `RunState`, `LoopState`, `Telemetry`, `AppContext` |
| `core_isr_events.h/cpp` | `core_` (L2) | ISR signaling via `volatile uint32_t` bitfield, `isrFetchAndClearEvents()` |
| `bsp_leds.h/cpp` | `bsp_` (L2) | LED control: init, green, blue, off |
| `bsp_flash.h/cpp` | `bsp_` (L2) | NRF_NVMC flash read/write primitives |
| `bsp_power.h/cpp` | `bsp_` (L2) | USB detection, DC-DC enable, System OFF sleep |
| `sensor_motion.h/cpp` | `sensor_` (L2) | LSM6DS3 motion detection (IMU object is file-static) |
| `comm_bthome.h/cpp` | `comm_` (L2) | BTHome v2 packet builder (pure logic, no HW deps) |
| `comm_ble_adv.h/cpp` | `comm_` (L2) | Bluefruit BLE advertising (**named `comm_ble_adv` NOT `ble` — Nordic SDK collision**) |
| `app_cli.h/cpp` | `app_` (L2) | AT command parser (8 commands) |
| `app_telemetry.h/cpp` | `app_` (L2) | Runtime stats: `adv_ms`, `tail_ms`, `idle_ms`, motion count, `[STATUS]` output |
| `app_config_store.h/cpp` | `app_` (L2) | Config load/save using `bsp_flash` primitives |
| `app_main.h/cpp` | `app_` (L3) | State machine orchestrator calling all modules |

Key design: `AppContext` struct bundles all mutable state (no globals), ISR event bitfield with atomic fetch-and-clear, file-static encapsulation, prefix-based naming for clear domain boundaries, no circular deps.

## Key Technical Details

### nRF52840 Sensor

- **Power modes**: Auto-detects USB vs battery via VBUS voltage. USB mode enables serial CLI + diagnostic output. Battery mode is production (deep sleep → wake → broadcast → sleep).
- **Motion detection**: LSM6DS3 wake-up interrupt on INT1 (GPIO P0.11). Threshold configurable 0x02-0x3F (value × 31.25mg). No activity state machine — uses wake-up pulse only.
- **USB motion detection**: INT1 uses latched interrupt (LIR=1). With RISING-edge ISR, if INT1 stays latched HIGH the ISR never re-fires. USB mode uses polling fallback: reads `WAKE_UP_SRC` register bit 3 (`WU_IA`) to detect motion when ISR misses edges. Boot clears stale latch + ISR events.
- **Broadcast cycle**: Wake → 300ms fast BLE advertising (20ms interval) → 3s tail window (listens for more motion) → sleep.
- **Sensitivity presets** in `config.h`: 0=custom, 1=high, 2=standard (default), 3=low. Affects threshold, tail window, TX power.
- **Flash storage**: Runtime config persisted to last 4KB page of nRF52840 flash (custom NRF_NVMC implementation). `RuntimeConfig` struct field order is FROZEN — binary compatibility with devices that already have flash-stored configs.
- **LED pins**: GPIO12 (blue), GPIO13 (green). Green flash = motion detected, blue flash = entering sleep.

### ESP32 Gateway

- **WiFiManager portal**: Creates `SeeedUA-Gateway` AP if unconfigured. Config stored in EEPROM.
- **MQTT topics published**: `bthome/{mac}/state` (sensor data), `gateway/{id}/info` (self-registration).
- **MQTT topics subscribed**: `gateway/{id}/cmd` (identify command triggers LED blink).
- **Device filtering**: Only processes BLE devices with name prefix `SeeedUA-`.

### FastAPI Backend

- **Product mapping**: `data/product_map.csv` — MAC address (lowercase, no colons) → SKU → product name → video file → screen ID.
- **Gateway registry**: `data/gateways.json` — auto-updated when gateways report info.
- **Deduplication**: Same MAC within `dedup_window` (default 2s) is ignored.
- **Sensor timeout**: Background thread checks every 1s. If no motion for `sensor_timeout` (default 5s), emits "put_down" event.
- **Event log**: In-memory ring buffer, last 100 events.
- **i18n**: Chinese (zh) and English (en), translation files in `app/backend/locales/`.
- **Threading**: Daemon threads for MQTT client loop and timeout checker, initialized in FastAPI lifespan context manager.
- **Threading**: Daemon threads for MQTT client loop and timeout checker, initialized in FastAPI lifespan context manager.

## MQTT Topic Reference

| Topic | Direction | Payload |
|-------|-----------|---------|
| `bthome/{mac}/state` | Gateway → Backend | `{"motion": bool, "rssi": int, "gateway_id": str}` |
| `screen/{id}/play` | Backend → Player | `{"video": str, "sku": str, "name": str}` |
| `gateway/{id}/info` | Gateway → Backend | `{"gateway_id", "mac", "ip", "board"}` |
| `gateway/{id}/cmd` | Backend → Gateway | `{"action": "identify"}` |

## Conventions

- MAC addresses are normalized to lowercase hex without colons (e.g., `f7dac49b63d1`).
- Commit messages follow conventional commits: `fix(sensor):`, `feat(gateway):`, `docs:`, `debug(sensor):`.
- The nRF52840 firmware uses a modular multi-file architecture with prefix-based naming: thin `.ino` entry point → `app_main.h/cpp` orchestrator → module files with domain prefixes (`core_*`, `bsp_*`, `sensor_*`, `comm_*`, `app_*`). Note: `comm_ble_adv.h` (not `ble.h`) to avoid name collision with Nordic SDK's `ble.h`.
- Both firmware projects use `config.h` for deployment-specific settings (credentials, sensitivity presets).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Love4yzp)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Love4yzp)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
