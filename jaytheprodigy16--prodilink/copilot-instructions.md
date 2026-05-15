## prodilink

> Open-source Python library for the Mudra Link wristband. Full device control over BLE, no proprietary software required.

# Prodilink

Open-source Python library for the Mudra Link wristband. Full device control over BLE, no proprietary software required.

## Tech Stack
- Python 3.10+, async-first
- bleak for BLE communication
- websockets for WebSocket server
- pytest + pytest-asyncio for tests

## Architecture
- `prodilink/protocol.py` - BLE UUIDs, enums, packet parsers
- `prodilink/commands.py` - Firmware command byte table and builder
- `prodilink/status.py` - Device status parser
- `prodilink/ble.py` - Low-level BLE connection management (with auto-reconnect)
- `prodilink/device.py` - High-level MudraLink API
- `prodilink/classifier.py` - IMU-based directional pinch classification
- `prodilink/server.py` - WebSocket server bridge for web UI
- `prodilink/web/index.html` - Self-contained web UI (vanilla JS, no build)
- `prodilink/cli.py` - CLI entry point (scan, info, gestures, serve, rebind)

## Conventions
- All BLE operations are async
- Firmware command bytes extracted from native binary (see re/firmware_commands.py)
- Protocol reference: re/protocol/PROTOCOL_REFERENCE.md
- Tests in tests/ directory, run with: pytest tests/ -v
- Web UI is a single self-contained HTML file (no npm, no build step)
- Logging via `logging` module: prodilink.ble, prodilink.device, prodilink.server

---
> Source: [JayTheProdigy16/Prodilink](https://github.com/JayTheProdigy16/Prodilink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
