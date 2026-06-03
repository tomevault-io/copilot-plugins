## geekhouse

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This is a MicroPython-based IoT server for Raspberry Pi Pico W that implements a HATEOAS-compliant REST API to control smart home devices (LEDs, motors, sensors, LCD display). The project includes both the MicroPython firmware code that runs on the device and a Python LLM-powered dialogue interface for natural language control.

## Architecture

The codebase is split into two main parts:

1. **MicroPython firmware** (`src/server/`): Runs on Raspberry Pi Pico W
   - `server.py`: Main server class that initializes Microdot web server, ConfigHandler, LCD, and Routes
   - `config_handler.py`: Loads JSON config and initializes Pin/ADC objects for devices
   - `routes.py`: Defines all REST API endpoints following HATEOAS principles (responses include `_links` navigation)
   - `utils.py`: WiFi connection and calibration utilities
   - `main.py`: Entry point that creates and runs IoTServer

1. **LLM dialogue controller** (`src/llm_iot/`): Python client application
   - `dialogue.py`: Uses Claude API to convert natural language commands into requests to the IoT server

## Device configuration system

Configuration flows from TOML → JSON → MicroPython objects:

1. Edit `config/config.toml` with device definitions (LEDs, sensors, motors)
1. Deploy script converts TOML to JSON using `deploy/toml_to_json.py`
1. `config_handler.py` loads JSON and creates `Pin` or `ADC` objects

Sensor calibration supports two types:

- `linear`: `calibrated = m * raw + b`
- `polynomial`: `calibrated = c[0] + c[1]*raw + c[2]*raw^2 + ...`

## Development workflow

### Setting up local development

```bash
# Create virtual environment and install dependencies
python -m venv venv
source venv/bin/activate  # or `venv\Scripts\activate` on Windows
pip install -r requirements.txt  # Generated from pyproject.toml using uv

# Or use uv (faster):
uv venv
source .venv/bin/activate
uv pip install -e .
```

### Deploying to Pico W

```bash
# 1. Edit configuration
cp config/config_example.toml config/config.toml
# Edit config.toml with your WiFi credentials and device pins

# 2. Deploy everything to device
./deploy/deploy.sh
```

The deployment script:

1. Converts `config.toml` to `config.json`
1. Installs MicroPython dependencies from `mp_requirements.txt` using `mpremote mip`
1. Copies library files from `lib/` to device
1. Copies all source files from `src/server/` to device root
1. Copies `config.json` to device
1. Resets device to start server

### Testing the API

```bash
# Test endpoints manually
curl http://DEVICE_IP/sensors
curl http://DEVICE_IP/leds
curl -X POST http://DEVICE_IP/leds/yellow_roof/toggle

# Or use the LLM dialogue interface
cd src/llm_iot
# Edit dialogue.py to set your device IP
python dialogue.py
# Then use natural language: "turn on the yellow light" or "what's the temperature?"
```

### Adding new devices

1. Add device configuration to `config/config.toml`
1. Run `./deploy/deploy.sh` to deploy updated config
1. API endpoints are automatically generated based on config

## Key implementation patterns

### HATEOAS response structure

All API responses follow this pattern (see `utils.create_response()`):

```json
{
  "data": { /* actual data */ },
  "_links": {
    "self": {"href": "/current/path"},
    "related": {"href": "/related/path"}
  }
}
```

### Dynamic route generation

Routes in `routes.py` are defined using Microdot decorators and dynamically reference devices from `config_handler`. Device IDs in URLs (like `/leds/{led_id}`) map to keys in the config dictionaries.

### Sensor calibration flow

1. ADC returns raw 16-bit value (0-65535)
1. `apply_calibration()` in `utils.py` applies configured transformation
1. API returns both raw and calibrated values

## Dependencies

### MicroPython (device)

- `microdot`: Web framework (installed via `mip` from GitHub)
- `lcd1602`: LCD driver (copied from `lib/`)

### Python (development machine)

- `mpremote`: Upload files and manage Pico W
- `anthropic`: Claude API client (for dialogue controller)
- `python-dotenv`: Environment variable management
- `requests`: HTTP client (for dialogue controller)

## File structure notes

- `src/server/`: MicroPython code deployed to device
- `src/llm_iot/`: Python client for LLM-powered control
- `lib/`: MicroPython libraries deployed to device
- `deploy/`: Deployment scripts and utilities
- `config/`: Device configuration (TOML source, JSON generated)
- `demo/`: Demo videos and recordings
- `docs/`: Documentation (includes Medium article draft)

---
> Source: [pavelanni/geekhouse](https://github.com/pavelanni/geekhouse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
