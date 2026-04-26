## unimixerserver

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UniMixerServer is a C# .NET 8.0 Windows service application that provides ESP32-S3 communication via serial interface and Windows audio session management. The application acts as a bridge between ESP32-S3 devices and Windows audio systems using a custom binary protocol over serial communication.

## Development Commands

### Building and Running
```bash
# Build the project
dotnet build

# Run the application
dotnet run

# Run with specific test modes
dotnet run -- --test-audio        # Test audio manager functionality
dotnet run -- --test-chrome       # Test Chrome icon conversion
dotnet run -- --debug-binary      # Debug binary protocol
dotnet run -- --test-ping         # Test ping functionality
dotnet run -- --desktop           # Launch desktop UI
```

## Project Structure
```
UniMixerServer/
├── Communication/           # Serial communication handlers
│   ├── BinaryProtocol/     # Binary protocol framing and CRC
│   ├── MessageProcessing/  # Message processors (JSON/Binary)
│   ├── SerialHandler.cs    # Serial communication handler
│   └── BaseCommunicationHandler.cs # Base communication interface
├── Configuration/          # Configuration management
├── Core/                   # Core audio management
│   ├── AudioManager.cs     # Windows audio session management
│   └── AudioSession.cs     # Audio session data models
├── Models/                 # Data models and message types
├── Services/               # Application services
│   ├── UniMixerService.cs  # Main background service
│   ├── AssetService.cs     # Process icon extraction and conversion
│   └── EspExceptionDecoder.cs # ESP32-S3 crash decoder
└── UI/                     # Desktop application launcher
```

## Key Technologies and Dependencies

- **.NET 8.0**: Target framework with Windows-specific features
- **System.IO.Ports**: Serial port communication
- **Serilog**: Structured logging with file rotation
- **Windows Audio APIs**: Native Windows audio session management
- **ESP-IDF Integration**: ESP32-S3 exception decoding with addr2line

## Configuration

Configuration is managed through `appsettings.json`:

### Key Configuration Sections
- **Audio**: Audio device filtering and session discovery
- **Serial**: Serial port settings and binary protocol configuration
- **Logging**: Comprehensive logging with categorized levels

### Serial Configuration
```json
"Serial": {
    "PortName": "COM17",
    "BaudRate": 115200,
    "DataBits": 8,
    "Parity": "None",
    "StopBits": "One",
    "EnableAutoReconnect": true,
    "BinaryProtocol": {
        "EnableBinaryProtocol": true,
        "EnableProtocolAutoDetection": true,
        "EnableDetailedLogging": true
    }
}
```

## Architecture Patterns

### Communication Architecture
- **Serial-Only**: Single communication channel via SerialHandler
- **Binary Protocol**: Custom binary framing with CRC16 validation
- **Message Processing**: O(1) message lookup using registered handlers
- **Event-Driven**: Communication handlers raise events for message processing

### Audio Session Management
- **Windows Audio API Integration**: Native Windows audio session enumeration
- **Process Icon Extraction**: Converts process icons to LVGL binary format
- **Real-time Monitoring**: Periodic refresh of audio session state
- **Volume Control**: Direct Windows audio session volume manipulation

### ESP32-S3 Exception Decoding
- **Automatic Detection**: Recognizes crash patterns in serial data
- **Modern RISC-V Support**: Uses ESP-IDF addr2line for accurate symbol resolution
- **Toolchain Discovery**: Automatically locates ESP-IDF installation
- **Crash Logging**: Saves decoded crashes with timestamps

## Message Protocol

### Message Types (String Constants)
```csharp
STATUS_UPDATE     // Audio session status broadcast
GET_STATUS        // Request device status
ASSET_REQUEST     // Request process icon data
SET_VOLUME        // Volume control command
PING_REQUEST      // Latency measurement
```

### Binary Protocol Structure
```
[START_BYTE][LENGTH_LSB][LENGTH_MSB][PAYLOAD][CRC_LSB][CRC_MSB][END_BYTE]
```

- **START_BYTE**: 0x7E (~)
- **LENGTH**: 16-bit little-endian payload length
- **PAYLOAD**: JSON message content
- **CRC**: CRC16 with polynomial 0xA001, init 0xFFFF
- **END_BYTE**: 0x7D (])

## Development Notes

### Audio Session Discovery
- Uses Windows Core Audio APIs for real-time session enumeration
- Supports filtering by process name, device type, and session state
- Automatically extracts and converts process icons to LVGL format

### ESP32-S3 Crash Decoding
- Requires firmware.elf file in `debug_files/` directory
- Automatically searches for ESP-IDF toolchain installation
- Supports both automatic detection and manual crash analysis
- Generates detailed crash reports with function names and line numbers

### Binary Protocol Debugging
- Use `--debug-binary` flag to analyze protocol frames
- Comprehensive CRC validation and frame structure analysis
- Supports testing with sample corrupted frames

### Logging Architecture
- Separate log files for different data types (incoming/outgoing/binary)
- Automatic log rotation with configurable size limits
- Real-time log monitoring with latest.log files
- Comprehensive performance and protocol statistics

## Testing and Debugging

### Audio Testing
```bash
dotnet run -- --test-audio
```
Tests Windows audio session enumeration and volume control.

### Icon Conversion Testing
```bash
dotnet run -- --test-chrome
```
Tests process icon extraction and LVGL binary conversion.

### Binary Protocol Testing
```bash
dotnet run -- --debug-binary
```
Analyzes binary protocol frames and CRC validation.

### Log Monitoring
```bash
# Windows batch files for log monitoring
logs/latest-unimixer.bat     # Main application logs
logs/latest-incoming.bat     # Incoming data logs
logs/latest-outgoing.bat     # Outgoing data logs
logs/latest-binary.bat       # Binary protocol logs
```

## Service Architecture

The application runs as a Windows background service with these main components:

1. **UniMixerService**: Main orchestration service managing timers and communication
2. **AudioManager**: Windows audio session management and monitoring
3. **SerialHandler**: Serial communication and binary protocol handling
4. **AssetService**: Process icon extraction and LVGL conversion
5. **EspExceptionDecoder**: ESP32-S3 crash analysis and symbol resolution

## Asset Processing

The application includes sophisticated asset processing for ESP32-S3 display integration:

- **Process Icon Extraction**: Extracts icons from running Windows processes
- **LVGL Conversion**: Converts icons to LVGL binary format using Node.js tools
- **Metadata Generation**: Includes dimensions, format, and checksum information
- **Caching**: Efficient caching to avoid redundant conversions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bezalel6) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
