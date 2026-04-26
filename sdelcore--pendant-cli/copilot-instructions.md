## pendant-cli

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Python tools for communicating with the Limitless Pendant BLE device. The protocol was reverse-engineered from the Limitless Android app (v2.1.6) using Google Protocol Buffers over BLE GATT.

## Commands

```bash
# Enter development environment (installs dependencies via uv)
nix develop

# Main operations
pendant status              # Device status (battery, recording, storage)
pendant info                # Device info (firmware, hardware, serial)
pendant download            # Download audio recordings
pendant download -d         # Download and decode to WAV
pendant decode input.opus   # Decode opus to wav

# Recording control
pendant record              # Start recording
pendant stop                # Stop recording

# Bluetooth management
pendant bt pair             # Pair with device (required first time)
pendant bt connect          # Connect to paired device
pendant bt disconnect       # Disconnect (keeps pairing)
pendant bt status           # Check connection status
pendant bt scan             # Scan for BLE devices

# Encryption
pendant crypto keygen       # Generate encryption keypair
pendant crypto setkey       # Send public key to device

# Debug
pendant explore             # Explore GATT services
pendant debug               # Download with detailed flash page inspection

# Regenerate protobuf (if proto/pendant.proto changes)
protoc --python_out=proto proto/pendant.proto
```

## Architecture

### Package Structure

```
src/pendant/
├── cli.py              # Main Click CLI entry point
├── commands/           # CLI command implementations
│   ├── device.py       # status, info
│   ├── audio.py        # download, decode, record, stop
│   ├── bluetooth.py    # bt subgroup
│   ├── crypto.py       # crypto subgroup
│   ├── explore.py      # GATT exploration
│   └── debug.py        # Debug download
└── core/               # Core library modules
    ├── config.py       # Constants, UUIDs, paths
    ├── protocol.py     # PendantProtocol class
    ├── dbus_client.py  # D-Bus backend
    ├── bleak_client.py # Bleak async backend
    ├── bluetooth.py    # bluetoothctl helpers
    ├── opus.py         # Opus decoding
    └── proto.py        # Protobuf import wrapper
```

### Communication Stack

```
pendant CLI  →  D-Bus/BlueZ  →  BLE GATT  →  Pendant Device
     ↓
PendantProtocol (protobuf encoding/decoding)
     ↓
DBusClient (notification handling, fragmentation)
     ↓
proto/pendant_pb2.py (generated protobuf classes)
```

### Key Components

**src/pendant/core/protocol.py** - Protocol implementation:
- BLE message framing with `PendantProtocol` class
- Fragment reassembly for large messages
- Audio extraction from FlashPage structures
- X25519 encryption key management

**proto/pendant.proto** - Protocol buffer definitions derived from decompiled Android app. Critical structure:
- `Chunk` is NOT a oneof - it has 13 independent optional fields
- `audio_data` is field 2, not field 1
- Audio comes in `codec_beamforming_data` (raw Opus frames)

**src/pendant/core/opus.py** - Handles raw Opus frames (no Ogg container). Uses TOC byte parsing for frame boundary detection.

### BLE UUIDs

| UUID | Purpose |
|------|---------|
| `632de002-...` | Control (write commands) |
| `632de003-...` | Data (receive notifications) |

### Audio Format

- Opus @ 16kHz mono, 16-bit
- Raw concatenated frames without Ogg container
- Use `pendant decode` - standard tools (ffmpeg, VLC) won't work

## Protocol Notes

- Device requires Bluetooth bonding before responding to any commands
- Messages use protobuf wrapped in BLE envelope for fragmentation
- Audio encrypted with X25519 + ChaCha20-Poly1305 (can inject own key)
- Device MAC set via `PENDANT_ADDRESS` env var (required, use `pendant bt scan` to find it)

See PROTOCOL.md for complete protocol documentation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sdelcore) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
