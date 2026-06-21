## homebridge-mhi-wfrac

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Homebridge plugin for Mitsubishi WF-RAC air conditioners controlled by the Smart M-Air app. The plugin exposes three HomeKit services: thermostat (for heating/cooling/auto), fan (for FAN mode), and dehumidifier.

## Development Commands

### Build and Development
- `npm run build` - Compiles TypeScript to JavaScript in the `dist/` directory
- `npm run watch` - Builds and links for development, then watches for changes using nodemon
- `npm run lint` - Runs ESLint with zero warnings tolerance
- `npm run prepublishOnly` - Runs lint and build (executed before publishing)

### Testing
There are no test scripts defined in this project.

## Code Architecture

### Core Components

**Entry Point (`src/index.ts`)**
- Registers the platform with Homebridge using the `PLATFORM_NAME` constant
- Single export function that takes the Homebridge API and registers the platform

**Platform (`src/platform.ts`)**
- `HomebridgeMHIWFRACPlatform` implements `DynamicPlatformPlugin`
- Manages device configuration and accessory lifecycle
- Reads device configs from `config.devices` array (name, mac, ip)
- Creates `WFRACAccessory` instances for each configured device
- Handles accessory restoration from cache using UUID generation based on MAC address

**Device Communication (`src/device.ts`)**
- `DeviceClient` handles HTTP communication with WF-RAC devices on port 51443
- `DeviceStatus` class manages device state parsing and serialization
- Complex binary protocol implementation for device communication
- Includes temperature lookup tables and operation mode mappings
- Advanced error handling and reconnection logic with exponential backoff
- Command queuing system for handling network failures

**Accessory Management (`src/platformAccessory.ts`)**
- `WFRACAccessory` creates and manages three HomeKit services per device:
  - Thermostat service (heating/cooling/auto modes)
  - Fan service (manual fan control)
  - Dehumidifier service (dry mode)
- Handles characteristic updates and user interactions
- Implements periodic status refresh every 10 seconds
- Maps device states to HomeKit characteristics and vice versa

### Configuration

**Plugin Configuration (`config.schema.json`)**
- Defines JSON schema for Homebridge Config UI X
- Requires `operatorId` (UUID from Smart M-Air app)
- Device array with name, MAC address, and IP address fields
- MAC address validation with regex pattern

**Settings (`src/settings.ts`)**
- Exports platform and plugin name constants
- Used for Homebridge platform registration

## Key Technical Details

### Device Protocol
- Uses base64-encoded binary protocol for device communication
- Temperature values use lookup tables for conversion
- Operation modes: 0=auto, 1=cool, 2=heat, 3=fan, 4=dry
- Airflow levels: 0=auto, 1=lowest, 2=low, 3=high, 4=highest

### HomeKit Integration
- Temperature range: 18-30°C with 0.5°C increments
- Fan speed mapping: 0%=auto, 25%=low, 50%=medium, 75%=high, 100%=highest
- Dehumidifier mode maps to device dry mode with auto temperature control

### Error Handling
- Sophisticated connection error categorization and recovery
- Exponential backoff for network failures
- Command queuing during connection issues
- Network health monitoring with periodic reconnection attempts

## TypeScript Configuration

- Target: ES2022 with ES module output
- Strict mode enabled with comprehensive type checking
- Source maps enabled for debugging
- Output directory: `dist/`

## Development Notes

- This is a platform plugin (not accessory plugin)
- Uses Homebridge dynamic platform architecture
- Requires network connectivity to WF-RAC devices
- Device discovery is manual via configuration (no auto-discovery)
- Operator ID must be obtained from Smart M-Air app registration process

---
> Source: [JobDoesburg/homebridge-mhi-wfrac](https://github.com/JobDoesburg/homebridge-mhi-wfrac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
