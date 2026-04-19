## si-faf-labs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
# Build active environment
pio run

# Build specific environment
pio run -e mega-freertos-4-1

# Upload to board
pio run -t upload

# Serial monitor
pio run -t monitor

# Build + monitor
pio run && pio run -t monitor
```

## Selecting Which Lab Compiles

The active lab is controlled by two mechanisms in `platformio.ini`:
1. The `[env:...]` section that is uncommented/active
2. The `build_flags = -DAPP_NAME=<N>` value (0–7)

`src/main.cpp` uses `APP_NAME` to conditionally call the matching `lab*AppSetup()` / `lab*AppLoop()`. Each `build_src_filter` also excludes irrelevant lab directories from compilation.

## Architecture

### Layers

```
Lab Application (lab0App → lab4-1App)
        ↓
Manager Components (taskManager, stateManager, sensorAcq, signalConditioning, alertManager, actuatorControl, userInput, reportDisplay)
        ↓
Hardware Abstraction (ddLed, ddButton, ddLcd, ddKeypad)
        ↓
Middleware (srvSerialStdio — enables printf/scanf over Serial)
        ↓
Arduino / FreeRTOS Framework
```

### Scheduler Evolution

| Labs | Scheduler |
|------|-----------|
| 0–1  | Blocking `loop()` |
| 2-1, 2-2 | Custom timer-based non-preemptive scheduler (ISR decrements counters, main loop checks `ready` flags) |
| 3-1, 3-2, 4-1 | FreeRTOS preemptive multitasking (`feilipu/FreeRTOS@^11.1.0-3`) |

### Key Conventions

- Every lab app exposes `lab*AppSetup()` and `lab*AppLoop()` matching Arduino's `setup()`/`loop()` contract.
- Hardware drivers (`dd*` modules) own all direct GPIO operations; manager/application layers must not access hardware directly.
- `srvSerialStdio` must be initialized before any `printf`/`scanf` usage.

## Simulation

Each lab has a `wokwi.toml` and `diagram.json` for circuit simulation via the Wokwi VSCode extension (no physical board required).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AlexDandy77) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
