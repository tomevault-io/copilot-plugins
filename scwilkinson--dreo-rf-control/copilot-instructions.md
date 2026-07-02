## dreo-rf-control

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Arduino project that controls a Dreo ceiling fan via RF signals using an ESP32-C6 and RFM69 radio module. The project implements an MQTT bridge for Home Assistant integration, allowing smart home control of both fan speed and light brightness.

**Status**: Production ready with full light and fan control functionality.

## Hardware Architecture

- **ESP32-C6**: Main microcontroller running the MQTT client and WiFi
- **RFM69HCW**: 433MHz radio module for transmitting RF commands to the Dreo fan
- **Pin Configuration**: Defined in `rfan.ino` around line 30
  - CS: D5, RST: D4, DIO2: D2, IRQ: D1

## Code Structure

### Core Files
- **rfan.ino**: Main Arduino sketch with MQTT client, WiFi setup, and command handling
- **RFM69Dreo.h/cpp**: Custom library encapsulating the RF protocol and radio control
- **commands/**: Directory containing RF signal captures and protocol documentation

### Key Components

#### RFM69Dreo Library
- Implements the proprietary Dreo RF protocol (433.92MHz, 3333 bps bitrate)
- Handles 10 different commands: light on/off, brightness up/down, fan on/off, fan speeds 1-6
- Protocol uses specific preamble, sync phrase, and command payloads (see `commands.txt`)

#### MQTT Integration
- **Two entities**: Separate light and fan entities in Home Assistant
- Light entity: Brightness control with 5 levels via up/down commands
- Fan entity: On/off toggle + 6-speed percentage control (0-100%)
- Topics follow Home Assistant convention: `homeassistant/light/{device_id}_light/` and `homeassistant/fan/{device_id}_fan/`
- Auto-discovery with proper JSON schema and templates

#### State Management  
- Tracks both light and fan states with persistence across reboots
- Light: on/off state + brightness (0-255 scale, 5 discrete levels: [51, 102, 153, 204, 255])
- Fan: on/off state + speed (0-6, mapped to percentages: 0%, 17%, 33%, 50%, 67%, 83%, 100%)
- **Critical**: RF is one-way communication - always assume commands succeed and update state accordingly

## Development Commands

This is an Arduino project - use Arduino IDE or PlatformIO for development:

- **Arduino IDE**: Open `rfan.ino` directly
- **Upload**: Use Arduino IDE's upload function or `arduino-cli compile` and `arduino-cli upload`
- **Serial Monitor**: 115200 baud for debugging output

## Protocol Details

The RF commands are stored as bit strings in `commands/commands.txt`. Each command consists of:
1. Preamble: `10001000100010001000100010001000`
2. Sync phrase: `11101110111011101000100011101110100011101000100010001000111011101000100010001000`  
3. Command payload: Varies by command (49 bits each)

The `RFM69Dreo` class abstracts this protocol - use the `Command` enum rather than raw bit strings.

## Configuration

WiFi and MQTT credentials are hardcoded in `rfan.ino` around lines 7-15. Production deployment uses these hardcoded values (user prefers this over dynamic configuration).

## Critical Issues & Solutions

### SPI Bus Management (CRITICAL BUG FIXED)
**Problem**: The RFM69Dreo library was calling `SPI.beginTransaction()` without corresponding `SPI.endTransaction()` calls, leaving the SPI bus locked and blocking WiFi/MQTT operations.

**Solution**: Added proper `SPI.endTransaction()` calls in:
- `RFM69Dreo::begin()` - after radio configuration
- `RFM69Dreo::sendCommand()` - wrap each command in begin/end transaction pair

**Symptom**: MQTT connection timeouts (error -4) despite successful TCP connections shown in broker logs.

### Fan Toggle Logic
**Problem**: Fan uses toggle-based RF commands, not absolute on/off like typical switches.

**Solution**: 
- For on/off commands: Only send toggle when state actually needs to change
- For speed commands: Always send the specific speed command (FAN_1 through FAN_6)
- Always update internal state after sending RF command (assume success)

### Home Assistant Integration Gotchas
1. **Buffer sizes**: Discovery JSON configs need large buffers (768+ bytes for light, 1024+ bytes for fan)
2. **State templates**: Fan entity requires `state_value_template` and `percentage_value_template` to parse JSON states
3. **Percentage parsing**: HA sends bare numbers (e.g., "75") which parse as JSON - handle both object and number formats
4. **Entity separation**: Light and fan must be separate entities with distinct topics

## User Preferences & Design Decisions

1. **No resyncing logic**: User handles state sync manually via physical remote if needed
2. **Production stability over convenience**: Hardcoded credentials, watchdog timer, health monitoring
3. **Always send RF commands**: Don't overthink toggle logic - send command and update state
4. **Minimal output**: Prefer concise responses, avoid lengthy explanations unless requested
5. **Direct implementation**: Focus on making it work rather than premature optimization

## Production Features
- Watchdog timer (60s timeout with auto-reboot)
- WiFi/MQTT reconnection with exponential backoff
- State persistence via ESP32 Preferences
- Memory leak monitoring (heap tracking)
- Availability heartbeat to Home Assistant
- Command debouncing (200ms)

## Debugging
- Serial monitor at 115200 baud
- Extensive MQTT debug output for troubleshooting
- State change logging with RF command confirmation

---
> Source: [scwilkinson/dreo-rf-control](https://github.com/scwilkinson/dreo-rf-control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
