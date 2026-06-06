## alpaca-driver

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

ZRO-Alpaca is a Go-based server implementing the ASCOM Alpaca REST API protocol for network control of astronomical devices. The project focuses on dome control with both simulated and real hardware drivers communicating via MQTT.

## Key Commands

### Build

```bash
# Build for Windows (default target)
make

# Build for current platform
go build -o tmp/main ./cmd/zro-alpaca

# Clean build artifacts
make clean
```

### Run

```bash
# Run the server
go run ./cmd/zro-alpaca -d

# Or after building
./tmp/main
```

### Test

```bash
# Run all tests
make test

# Run specific test package
go test ./pkg/drivers/zro/...

# Run specific test
go test -run TestParseResponse ./pkg/drivers/zro/...
```

### Development

```bash
# Format code
go fmt ./...

# Check for issues (no linter configured - use standard go vet)
go vet ./...
```

## Architecture

### Core Components

1. **Alpaca Protocol Layer** (`/pkg/alpaca/`)

   - `device.go`: Base device interface all drivers must implement
   - `dome.go`: Dome-specific Alpaca API endpoints
   - `server.go`: HTTP server handling Alpaca REST requests
   - `discovery.go`: Device discovery protocol implementation
   - `store.go`: BoltDB persistence layer for device configurations

2. **Alpaca Drivers** (`/pkg/drivers/`)

   - `/zro/`: Real ZRO dome driver using MQTT for hardware communication
   - `/dome_simulator/`: Simulated dome for testing without hardware

3. **Dome Driver** (`/pkg/dome/`)

   - Actual dome control logic. No alpaca-specific code here.
   - Handles azimuth slewing, parking, and status reporting
   - Uses MQTT for communication with hardware

4. **MQTT Communication**

   - Uses `paho.mqtt.golang` client
   - Command topic: `/ZRO/commands`
   - Response topic: `/ZRO/responses`
   - Protocol: JSON messages with action/response patterns

5. **Web UI** (`/templates/`)
   - Embedded HTML templates for device setup
   - Accessible at `http://localhost:8090/api/v1/dome/1/setup`

### Key Patterns

- All device drivers implement the `Device` interface from `device.go`
- HTTP handlers follow Alpaca REST API specification
- MQTT messages use JSON with specific action/response formats
- Configuration stored in BoltDB (`alpaca.db`)
- Extensive use of structured logging via logrus

## Environment Configuration

Required for MQTT hardware communication:

- `MQTT_BROKER`: MQTT broker address (default: `tcp://localhost:1883`)
- `MQTT_USERNAME`: MQTT authentication username
- `MQTT_PASSWORD`: MQTT authentication password
- `ALPACA_PORT`: Server port (default: 8090)

## Testing Approach

- Unit tests alongside source files (`*_test.go`)
- Uses `testify` for assertions
- Focus on protocol parsing and hardware response handling
- Run simulator for integration testing without hardware

---
> Source: [CerecedaObs/alpaca-driver](https://github.com/CerecedaObs/alpaca-driver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
