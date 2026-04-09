## paradox-dgp-mqtt

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a .NET 9 service that communicates with Paradox Digiplex/EVO alarm panels via serial connection and exposes the panel state through MQTT. It was converted from a Python implementation.

## Build Commands

```bash
# Build all projects
dotnet build

# Run the worker service
dotnet run --project Digiplex.Worker

# Publish for Linux deployment
dotnet publish Digiplex.Worker -c Release -r linux-x64 --self-contained
```

## Architecture

### Projects

- **Digiplex.Core**: Shared library containing models, protocol codec, and observable state
- **Digiplex.Serial**: Serial communication with the panel
- **Digiplex.Mqtt**: MQTT integration with Mosquitto broker
- **Digiplex.Worker**: .NET hosted service entry point

### Key Components

1. **DigiplexCodec** (`Digiplex.Core/Protocol/`): Encodes/decodes binary PDUs for serial communication
2. **DigiplexState** (`Digiplex.Core/Services/`): Observable state container using System.Reactive
3. **SerialService** (`Digiplex.Serial/`): Handles panel connection, login, and polling loop
4. **MqttService** (`Digiplex.Mqtt/`): Publishes state changes and subscribes to commands

### Protocol Flow

1. Send init string → receive InitPdu
2. Send login (InitPdu with password) → receive LoginResponsePdu
3. Load zone labels from EEPROM (0x2000-0x22F0)
4. Poll zone status from RAM (0x8153) every second

### PDU Types (in `Digiplex.Core/Models/Pdu/`)

- `InitPdu` / `LoginResponsePdu` - Handshake and authentication
- `ReadRequestPdu` / `ReadResponsePdu` - Memory read operations
- `WriteRequestPdu` / `WriteResponsePdu` - Memory write operations
- `MonitorRequestPdu` / `MonitorResponsePdu` - Partition control
- `EventRequestPdu` / `EventResponsePdu` - Event log retrieval
- `ErrorResponsePdu` - Error responses from panel

### Memory Addresses

- EEPROM: address & 0x7FFF
- RAM: address | 0x8000
- Zone labels: EEPROM 0x2000-0x22F0 (16 bytes each)
- Zone status: RAM 0x0153 (6 bytes zone + 6 bytes tamper)

## Configuration

Settings are in `Digiplex.Worker/appsettings.json`. Key sections:

- `Serial`: Port name, baud rate, password, poll interval
- `Mqtt`: Broker host/port, topic prefix, Home Assistant discovery

## Future Work

- Implement arm/disarm commands via MQTT (handle messages on `digiplex/partition/{id}/arm/set`)
- Add partition status polling
- Event log retrieval and publishing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adumat)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/adumat)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
