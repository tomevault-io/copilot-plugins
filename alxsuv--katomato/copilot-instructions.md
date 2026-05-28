## katomato

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Katomato is a modular indoor plant automation system with two codebases: a Python control app (`app/`) and Arduino firmware (`arduino/`). The Python app receives sensor JSON from Arduino over serial, runs control logic, and sends actuator commands back. InfluxDB stores time-series sensor data.

## Commands

### Python App (from repo root)

```bash
make install            # Install Python dependencies (Poetry)
make test               # Run all pytest tests
make lint               # Run ruff linter
make run                # Run the app (requires Arduino connected)

# Run a single test file
cd app/src && poetry run pytest tests/controllers/test_temperature.py

# Run a specific test
cd app/src && poetry run pytest tests/controllers/test_temperature.py::test_name -v
```

### Arduino Firmware

```bash
make build-arduino      # Build firmware
make upload-arduino     # Flash to Arduino UNO
make monitor-arduino    # Open serial monitor
```

### Docker (InfluxDB)

```bash
make up                 # Start InfluxDB container
make down               # Stop InfluxDB container
```

## Architecture

### Registration Pattern (core design pattern)

Controllers and devices self-register via decorators in `core/registry.py`:
- `@controller_registry("sensor_key")` — maps a sensor name (e.g., `"dt"` for temperature) to a controller class
- `@device_registry("device_name")` — maps a device name (e.g., `"exhaust_fan"`) to a device class

Both `controllers/__init__.py` and `devices/__init__.py` use `pkgutil` to auto-import all modules in their package, triggering decorator registration at startup. To add a new controller or device, create a new module in the respective package — no other wiring needed.

### Data Flow

1. Arduino sends JSON: `{"sensor": "dt", "label": "Temperature", "value": 25.3, "unit": "°C", "controls": [{"pin": 4, "type": "digital", "device": "exhaust_fan"}]}`
2. `core/serial.py` receives and parses into `SensorData` (defined in `core/sensor_data.py`)
3. `core/dispatcher.py` → `controller_dispatcher` looks up the controller by `sensor_data.sensor` key
4. Controller evaluates the reading against config thresholds, then calls registered devices with `Action.UP` or `Action.DOWN`
5. Devices enqueue JSON commands → `command_dispatcher` sends them back to Arduino over serial

### Configuration & Growth Phases

- `config/base.py` defines `BaseConfig` with serial, InfluxDB, and growth-phase settings
- Plant profiles (e.g., `config/zucchini.py`) extend `BaseConfig` and define growth phases as inner dataclasses with thresholds (temp, humidity, moisture ranges)
- Active profile is selected by `APP_PROFILE` env var (defaults to `"zucchini"`)
- Growth phases can be switched at runtime via CLI command `{"command": "phase", "name": "PhaseName"}`

### Arduino Side (`arduino/`)

- `src/main.cpp` — entry point, reads sensors in loop, dispatches serial commands
- `src/sensors/` — sensor reading modules
- `src/controls/` — actuator control modules (fans, relays, etc.)
- `src/communication/` — serial protocol handling
- `src/config/` — pin assignments and hardware constants
- Built with PlatformIO targeting Arduino UNO (`atmelavr`)

### Testing

Tests mirror the source structure under `app/src/tests/`. The shared `conftest.py` provides a `ctx` fixture with a mock command queue and mock device pre-registered in `DEVICE_REGISTRY`. Tests use `pytest-asyncio` for async controller/device tests. Hardware is fully mocked — no Arduino needed to run tests.

---
> Source: [alxsuv/katomato](https://github.com/alxsuv/katomato) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
