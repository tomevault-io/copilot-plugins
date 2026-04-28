## openmv-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Install Dependencies

### macOS

```bash
brew install cmake llvm openssl
export PATH="$(brew --prefix llvm)/bin:$PATH"  # add to ~/.zshrc for clang-tidy/clang-format
```

### Ubuntu / Debian

```bash
sudo apt install cmake clang-tidy clang-format libssl-dev
```

### Windows

```powershell
winget install Kitware.CMake LLVM.LLVM ShiningLight.OpenSSL.Dev
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\LLVM\bin", "Machine")
# restart terminal
```

## Download Resources

```bash
curl -L -o /tmp/openmv-ide-resources.zip https://github.com/openmv/openmv-ide/releases/download/v4.8.9/openmv-ide-resources-4.8.9.zip
unzip -o /tmp/openmv-ide-resources.zip -d resource
rm /tmp/openmv-ide-resources.zip
```

## Build & Run

```bash
# First time: git submodule update --init --recursive
cmake -B build
cmake --build build

# Run server (default port 15257). Resources are embedded in the binary
# and extracted to a temp directory on first launch.
./build/openmv_mcp_server
./build/openmv_mcp_server --port 9000
```

## Test

```bash
cmake --build build
ctest --test-dir build --output-on-failure
```

## Code Quality

clang-tidy runs automatically during compilation (`CMAKE_CXX_CLANG_TIDY`). Disable with `-DENABLE_CLANG_TIDY=OFF`.

```bash
cmake --build build --target format   # auto-format with clang-format
cmake --build build --target check    # check formatting (CI)
```

## Architecture

This is an MCP (Model Context Protocol) server for controlling OpenMV cameras. It uses **Streamable HTTP** transport (not stdio) over JSON-RPC 2.0.

Resources (firmware, dfu-util/sdphost/blhost/stcubeprogrammer/mpy-cross for the
host platform) are packaged into the binary at build time via CMakeRC and
extracted at runtime to `$TMPDIR/openmv-mcp-<crc>/` (idempotent across runs).

### Core Modules

- **MCP Server**: `src/server/mcp_server.h/.cpp` — HTTP routing, JSON-RPC dispatch, WebSocket endpoints
- **MCP Tool**: `src/server/mcp_tool.h/.cpp` — tool definitions (`ALL_MCP_TOOLS` vector) and tool call dispatch
- **MCP Context**: `src/server/mcp_context.h` — camera instance management, streaming message support
- **Camera**: `src/camera.h/.cpp` — abstract camera class with factory pattern (`Camera::create()`) and callback system (connected/script/terminal/frame)
- **Frame**: `src/frame.h/.cpp` — frame data with pixel format conversion, JPEG encoding via stb
- **Protocol**: `src/protocol/` — OpenMV protocol v1 (opcode-based) and v2 (packet-based with CRC)
- **Serial Port**: `src/serial_port/` — cross-platform serial I/O (macOS/Linux/Windows)
- **Subprocess**: `src/subprocess/` — cross-platform process execution with output callback streaming and cancellation support
- **Camera List**: `src/camera_list/` — platform-specific USB camera discovery
- **MCP Client**: `src/client/mcp_client.h/.cpp` — MCP protocol client for testing server interactions
- **Firmware**: `src/firmware.h/.cpp` — firmware flash and repair operations using subprocess-based flashing tools
- **Info**: `src/info.h/.cpp` — system information and license management for OpenMV cameras
- **Board**: `src/board.h/.cpp` — board/sensor database, USB device lookup, and firmware command definitions
- **Daemonize**: `src/daemonize.h/.cpp` — POSIX background process fork with stdout/stderr redirection
- **Resource**: `src/resource.h/.cpp` — extract CMakeRC-embedded resources (firmware, flashing tools) to a temp directory at runtime
- **Utilities**: `src/utils/` — base64, CRC, ring buffer, UTF-8 buffer

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SingTown) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
