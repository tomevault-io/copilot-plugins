## varsigns

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a PlatformIO-based embedded systems project targeting the NodeMCU v2 (ESP8266) development board. The project is configured to control BTF-LIGHTING 2835SMD WS2811 IC RGB addressable LED strips (DC24V, 108LED/m, 18Pixels/m).

## Hardware Configuration

- **Target Board**: NodeMCU v2 (ESP8266)
- **Platform**: Espressif8266
- **Framework**: Arduino
- **LED Strip**: BTF-LIGHTING 2835SMD WS2811 IC RGB Chasing Color LED Strip
  - 16.4FT length
  - DC24V
  - 108 LEDs/meter
  - 18 Pixels/meter
  - 8mm width
  - Addressable RGB with WS2811 IC

## Development Commands

### Building
```bash
pio run
```

### Uploading to Device
```bash
pio run --target upload
```

### Serial Monitor
```bash
pio device monitor
```

### Clean Build
```bash
pio run --target clean
```

### Build and Upload
```bash
pio run --target upload && pio device monitor
```

## Project Structure

- `src/main.cpp` - Main Arduino sketch with `setup()` and `loop()` functions
- `platformio.ini` - PlatformIO configuration file defining board and platform
- `lib/` - Project-specific libraries
- `include/` - Header files for the project
- `test/` - Unit tests
- `.pio/` - PlatformIO build artifacts (gitignored)

## Architecture Notes

This is an Arduino-based embedded project using the PlatformIO build system. Code follows the standard Arduino structure with `setup()` running once at startup and `loop()` running continuously.

When working with the WS2811 LED strip, you'll likely need to use a library like FastLED or Adafruit NeoPixel that supports the WS2811 protocol. The LED strip operates at DC24V and has 18 addressable pixels per meter.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aaronwald) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
