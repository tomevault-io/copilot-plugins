## candelta

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CANDelta is a real-time CAN bus monitor for reverse engineering vehicle networks. It consists of:
- **Firmware** (C): RP2040-based CAN analyzer using Pico SDK
- **Desktop App** (C#): Cross-platform Avalonia UI application

The Monitor view displays one row per unique CAN ID with 8 data bytes. Each byte has color-coded highlighting that flashes on value changes and fades back to idle, helping identify active signals during reverse engineering.

## Project Structure

```
CANDelta/
├── firmware/                       # RP2040 Pico SDK firmware (C)
│   ├── CMakeLists.txt
│   ├── pico_sdk_import.cmake
│   ├── include/
│   │   └── candelta_config.h       # Pin assignments, CAN speeds, buffer sizes
│   └── src/
│       ├── main.c                  # Dual-core entry point
│       ├── can/
│       │   ├── mcp2515.h
│       │   └── mcp2515.c           # MCP2515/MCP25625 SPI driver
│       ├── usb/
│       │   ├── usb_comm.h
│       │   └── usb_comm.c          # USB CDC + thread-safe ring buffer
│       └── protocol/
│           ├── commands.h
│           └── commands.c          # Wire protocol command handler
│
├── src/
│   ├── CANDelta.Core/              # Shared .NET library
│   │   ├── Protocol/
│   │   │   ├── CanFrame.cs         # CAN frame struct with signatures
│   │   │   ├── Commands.cs         # Protocol enums and types
│   │   │   └── ProtocolParser.cs   # Wire protocol serialization
│   │   ├── Models/
│   │   │   ├── Behaviour.cs        # Control/test behaviour container
│   │   │   ├── Trace.cs            # Single capture session
│   │   │   └── DeltaResult.cs      # Analysis output
│   │   └── Analysis/
│   │       └── DeltaAnalyzer.cs    # Delta computation with noise filtering
│   │
│   ├── CANDelta.App/               # Avalonia UI application
│   │   ├── App.axaml(.cs)
│   │   ├── ViewModels/
│   │   │   ├── MainWindowViewModel.cs
│   │   │   ├── MonitoredByte.cs    # Per-byte value tracking and color animation
│   │   │   ├── MonitoredCanId.cs   # Row model with 8 bytes
│   │   │   └── ColorTheme.cs       # Theme presets for highlighting
│   │   ├── Views/
│   │   │   └── MainWindow.axaml(.cs)
│   │   └── Services/
│   │       ├── ISerialService.cs
│   │       └── SerialService.cs    # USB CDC async communication
│   │
│   └── CANDelta.Desktop/           # Desktop entry point (Win/Linux/Mac)
│       └── Program.cs
│
├── tests/
│   └── CANDelta.Core.Tests/        # xUnit tests
│       ├── DeltaAnalyzerTests.cs
│       └── ProtocolParserTests.cs
│
├── docs/
│   └── protocol.md                 # Wire protocol specification
│
├── CANDelta.sln
├── .gitignore
├── README.md
└── CLAUDE.md
```

## Build Commands

### Firmware (Pico SDK)
```bash
cd firmware
mkdir build && cd build
cmake .. -DPICO_SDK_PATH=/path/to/pico-sdk
make
```
Output: `candelta.uf2` - flash by holding BOOTSEL while connecting USB.

### .NET Application
```bash
# Build all projects
dotnet build

# Run desktop app
dotnet run --project src/CANDelta.Desktop

# Run tests
dotnet test

# Run specific test
dotnet test --filter "FullyQualifiedName~DeltaAnalyzerTests.Analyze_FindsUniqueTestFrames"
```

## Architecture

### Firmware (firmware/)
Dual-core architecture:
- **Core 0**: USB CDC communication, command processing (`main.c`, `protocol/commands.c`)
- **Core 1**: CAN frame capture with microsecond timestamps (`main.c` core1_entry)

Key modules:
- `can/mcp2515.c` - MCP2515/MCP25625 SPI driver
- `usb/usb_comm.c` - Thread-safe ring buffer for Core 1 → Core 0 frame transfer
- `protocol/commands.c` - Wire protocol command handler

Pin assignments in `include/candelta_config.h` match Adafruit Feather RP2040 CAN.

### .NET Projects

**CANDelta.Core** - Shared library (no UI dependencies)
- `Protocol/` - Wire protocol types and parser (mirrors firmware protocol)
- `Models/` - Domain models: Behaviour, Trace, DeltaResult
- `Analysis/DeltaAnalyzer.cs` - Delta computation with noise threshold filtering

**CANDelta.App** - Avalonia UI (MVVM with CommunityToolkit.Mvvm)
- `Services/SerialService.cs` - USB CDC communication with async command/response
- `ViewModels/MainWindowViewModel.cs` - Monitor logic with frame batching and animations
- `ViewModels/MonitoredByte.cs` - Per-byte value/min/max tracking and color transitions
- `ViewModels/MonitoredCanId.cs` - Row model with 8 MonitoredByte instances
- `ViewModels/ColorTheme.cs` - Selectable themes (Cyan, Orange, Lime, etc.)
- `Views/` - AXAML views with ItemsControl and VirtualizingStackPanel

**CANDelta.Desktop** - Entry point for Windows/Linux/macOS

### Wire Protocol
Binary protocol over USB CDC. See `docs/protocol.md` for full specification.
- Packets framed with STX (0x02) / ETX (0x03)
- Commands: PING, START_CAPTURE, STOP_CAPTURE, SET_SPEED, etc.
- CAN frames streamed async with 64-bit microsecond timestamps

### Monitor Architecture
The app uses high-performance patterns for real-time visualization:
1. **Frame batching**: ConcurrentQueue receives frames from serial thread, UI dequeues in batches (100/tick)
2. **Dirty tracking**: HashSet tracks IDs with active animations, only those get fade updates
3. **Brush mutation**: Existing SolidColorBrush colors are mutated instead of creating new instances
4. **Binary insertion**: New CAN IDs inserted in sorted order with O(log n) binary search
5. **Color intensity**: Per-byte min/max tracking; change intensity = delta / range

## Key Patterns

- **Avalonia**: Use `.axaml` extension, `x:DataType` for compiled bindings
- **MVVM**: ViewModels use `[ObservableProperty]` and `[RelayCommand]` attributes
- **Thread safety**: UI updates via `Avalonia.Threading.Dispatcher.UIThread.Post()`
- **Firmware SPI**: CS manually controlled via GPIO (not hardware CS)

## Hardware

- Adafruit Feather RP2040 CAN (MCP25625 = MCP2515 + TJA1042 transceiver)
- 16MHz crystal for CAN timing calculations
- SPI1 on pins: MISO=12, MOSI=15, SCK=14, CS=9, INT=11

## Git Commit Rules

- Do NOT add AI attribution (emoji, "Generated with Claude", "Co-Authored-By" lines, etc.) to commit messages
- Keep commit messages concise and focused on the changes made

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mitsosops) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
