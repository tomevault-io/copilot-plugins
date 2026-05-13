## kv

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

KV is a modern remote KVM (Keyboard, Video, Mouse) solution built in Crystal that provides ultra low-latency video/audio streaming from a server's HDMI output bidirectional input via USB gadget framework. It operates in three modes:

- **Full KVM Mode**: Video + Audio + Input + Storage (requires USB OTG port)
- **Video Streaming Mode**: Video + Audio only (no USB hardware required)
- **Input Mode**: Keyboard/mouse only via USB gadget (no video capture required)

## Development Commands

### Building
```bash
# Install dependencies
shards install

# Development build (single binary)
crystal build --release src/main.cr -o bin/kv

# Build static binaries for distribution (AMD64 and ARM64)
./build_static.sh
```

### Testing
```bash
# Run tests (currently has issues with libcomposite kernel module loading at module level)
crystal spec

# Run specific test
crystal spec spec/kv_spec.cr
```

### Linting
```bash
# Run linter
ameba src/ spec/

# Auto-fix linting issues (preferred before commits)
ameba --fix src/ spec/
```

## Architecture

### Core Components

The application follows a centralized orchestration pattern with `GlobalKVM` managing component lifecycle:

- **KVM Manager** (`src/kvm_manager.cr`): Central orchestrator managing all KVM components, global state, and lifecycle
- **Video Capture** (`src/video_capture.cr`): V4L2-based MJPEG streaming using `v4cr` library with direct device access
- **Audio Streaming** (`src/audio_streamer.cr`): ALSA capture with Opus encoding for efficient streaming
- **USB Composite Gadget** (`src/composite.cr`): Manages HID (keyboard/mouse), ECM (ethernet), and mass storage USB functions
- **Input Handling** (`src/keyboard.cr`, `src/mouse.cr`): HID event processing with duplicate prevention
- **WebSocket Manager** (`src/websocket_manager.cr`): Real-time client communication and status broadcasting
- **Hardware Detector** (`src/hardware_detector.cr`): Validates USB OTG and video device availability

### Web Interface Architecture

- **HTTP Server**: Kemal-based with WebSocket support and optional basic auth (`KV_USER`, `KV_PASSWORD`)
- **API Endpoints** (`src/endpoints/`): Modular REST API for each functional area:
  - `main.cr`: Web interface rendering
  - `video.cr`: Resolution, quality, streaming control
  - `input.cr`: Keyboard/mouse event handling
  - `audio.cr`: Audio streaming control
  - `storage.cr`: Disk image upload and USB attachment
  - `ethernet.cr`: USB network interface management
  - `status.cr`: System status monitoring
- **Static Assets**: Embedded via baked_file_system, responsive design with mobile templates

### Key Implementation Details

#### Video Processing
- **Direct V4L2 access** for minimal latency
- **MJPEG streaming** with quality adjustment (1-100%)
- **Hotplug support** with configurable polling interval
- **Resolution detection** from device capabilities
- **Frame rate monitoring** and statistics

#### USB Gadget Framework
- **libcomposite kernel module** requirement (auto-loaded if available)
- **Multi-function gadget**: HID + ECM + Mass Storage
- **HID devices**: Keyboard (boot protocol) and Mouse (relative/absolute)
- **Mass Storage**: Exposes disk images as virtual USB drives
- **System cleanup** on startup/shutdown

#### Input Device Management
- **Duplicate prevention**: Tracks pressed keys/mouse buttons
- **Anti-idle support**: Optional mouse jiggler every 60 seconds
- **Event queuing**: Prevents event flooding

### CLI Structure

Uses docopt with comprehensive options:
- Device selection (`-d`, `-a`): Video/audio device paths
- Video controls (`-r`, `-f`, `-q`): Resolution, framerate, quality
- Server config (`-p`, `-b`): Port and bind address
- Feature toggles (`--disable-*`): Selective gadget disabling
- System options (`--anti-idle`, `--hotplug-interval`): Behavioral controls

## Important Development Notes

### Code Style Requirements
- **No `not_nil!` usage** - prefer nil-safe handling patterns
- **Descriptive parameter names** in blocks (avoid single letters)
- **Docopt CLI** for command line interfaces
- **Pico.css** for web interface styling

### Quality Assurance
- **Build verification**: Always test with `crystal build --release src/main.cr -o bin/kv`
- **Binary verification**: Check `./bin/kv --help` runs correctly
- **Linting**: Run `ameba --fix src/ spec/` before considering changes complete
- **Single binary focus**: The project produces one main binary (`kv`)

### Dependencies & External Code
- **Managed via shards.yml**: All Crystal dependencies
- **lib/ directory**: External libraries (never modify)
- **Key dependencies**: kemal, v4cr, opus, baked_file_system, docopt, pluto

### Hardware Requirements
- **USB OTG port**: Required for full KVM functionality
- **V4L2 MJPEG device**: Most cheap HDMI capture dongles work
- **Linux with libcomposite**: Required kernel module for USB gadget

### Runtime Requirements
- **Root privileges**: Required for USB gadget functionality and hardware access
- **Kernel modules**: libcomposite must be available (auto-loaded when possible)
- **Device permissions**: Access to /dev/video* and USB gadget endpoints

### Test Environment Notes
- **libcomposite loading issue**: Tests fail due to kernel module loading at module level (line 215 in main.cr)
- **Hardware dependent**: Video capture tests require actual hardware devices

## Build Verification Workflow

Always verify changes by:
1. **Install dependencies**: `shards install` (if needed)
2. **Build binary**: `crystal build --release src/main.cr -o bin/kv`
3. **Test help output**: `./bin/kv --help`
4. **Run linter**: `ameba --fix src/ spec/`
5. **Check for linting errors**: All should pass

---
> Source: [ralsina/kv](https://github.com/ralsina/kv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
