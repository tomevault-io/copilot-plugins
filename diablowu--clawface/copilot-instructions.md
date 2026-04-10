## clawface

> ESP8266 (NodeMCU V2) based multifunctional desktop display showing time, weather, temperature/humidity, and animations. Arduino/PlatformIO project for embedded systems with limited RAM (80KB) and Flash (4MB).

# AGENTS.md - SmallDesktopDisplay Development Guide

## Project Overview

ESP8266 (NodeMCU V2) based multifunctional desktop display showing time, weather, temperature/humidity, and animations. Arduino/PlatformIO project for embedded systems with limited RAM (80KB) and Flash (4MB).

## Build Commands

### Build Operations
```bash
pio run                      # Build firmware (compiles without uploading)
pio run -e esp12e            # Build for specific environment
pio run --target clean       # Clean build
pio run --target rebuild    # Clean + build
pio run --target size        # Size analysis (monitor RAM usage)
```

### Upload/Flash
```bash
pio run --target upload --upload-port /dev/ttyUSB0
pio run --target upload --upload-port /dev/ttyUSB0 --upload-speed 3000000
esptool.py --port /dev/ttyCH341USB0 --baud 3000000 write_flash 0x00000 .pio/build/esp12e/firmware.bin
```

### Monitor/Debug
```bash
pio device monitor --port /dev/ttyCH341USB0 --baud 115200
pio device list             # List connected devices
```

### Testing
PlatformIO test framework is configured but not actively used. The `test/` directory contains font/animation assets, not unit tests:
```bash
pio test                    # Run tests (if implemented)
pio test -f filename        # Run specific test file
pio test -v                 # Verbose mode
```

## Code Style Guidelines

### General Principles
- Memory-efficient code; avoid dynamic allocation when possible
- Keep functions small and focused; use early returns
- Code must compile for ESP8266 @ 80MHz

### File Organization
- Main logic: `src/main.cpp`
- Configuration: `src/config.h`
- Modules: `src/Animate/`, `src/weatherNum/`, `src/wifiReFlash/`
- Assets: `font/`, `img/` directories

### Naming Conventions
| Type | Convention | Example |
|------|------------|---------|
| Files | lowercase with underscores | `wifi_re_flash.cpp` |
| Classes | PascalCase | `TFT_eSPI` |
| Functions | camelCase | `getCityWeather` |
| Variables | camelCase | `cityCode`, `updateWeatherTime` |
| Constants/Macros | UPPER_SNAKE_CASE | `LCD_BL_PIN`, `WM_EN` |
| Structs | PascalCase | `config_type` |

### Code Formatting
- 2 spaces for indentation (no tabs)
- Opening brace on same line (K&R style)
- Spaces around operators: `a + b`, not `a+b`
- Max line length: 120 characters
- Add Chinese comments for user-facing code

### Import Order
1. Standard library (`<Arduino.h>`, `<SPI.h>`)
2. Framework headers (`<TFT_eSPI.h>`, `<EEPROM.h>`)
3. Local headers (`"config.h"`, `"font/timeClockFont.h"`)

### Header Files
- Use include guards: `#ifndef`, `#define`, `#endif`
- Use forward declarations to reduce dependencies

### Types and Variables
- Use fixed-width types: `uint8_t`, `uint16_t`, `uint32_t`
- Prefer `int` for loop counters
- Use `float` for temperature/humidity
- Use `String` carefully; avoid excessive concatenation
- Initialize variables at declaration

### Error Handling
- Check return values of network operations
- Validate input ranges (brightness 0-100, rotation 0-3)
- Use `Serial.println` for debugging
- Handle WiFi connection failures gracefully

## Configuration (config.h)

| Macro | Default | Description |
|-------|---------|-------------|
| Animate_Choice | 2 | Animation: 1=astronaut, 2=Hutao |
| TMS | 1000 | Time base (ms) |
| WM_EN | 1 | WiFi Manager (WEB config) enabled |
| DHT_EN | 0 | DHT11 sensor enabled |

## Hardware Pin Definitions

| Pin | GPIO | Function |
|-----|------|----------|
| SCK | 14 | SPI Clock |
| MOSI | 13 | SPI MOSI |
| RES | 2 | LCD Reset |
| DC | 0 | LCD Data/Command |
| LCDBL | 5 | LCD Backlight (PWM) |
| DHT11 | 12 | DHT11 sensor (optional) |
| Button | 4 | Input button |

## Dependencies
- DHT sensor library, ArduinoJson, Button2, FastLED
- TJpg_Decoder, Thread, TFT_eSPI

## Memory Optimization
- Use PROGMEM for read-only data (fonts, images)
- Avoid recursion; reuse sprite buffers
- Put large constant arrays in Flash

## Serial Commands (Debug/Config)
| Command | Function |
|---------|----------|
| 0x01 | Set brightness (0-100) |
| 0x02 | Set city code (9 digits) |
| 0x03 | Set screen rotation (0-3) |
| 0x04 | Set weather update interval |
| 0x05 | Reset WiFi settings |

## Screen Orientation
- 0: USB connector down, 1: right, 2: up, 3: left

## Version
- Current: SDD V1.4.3, Target: ESP8266 (ESP-12E), Flash: 4MB

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/diablowu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/diablowu)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
