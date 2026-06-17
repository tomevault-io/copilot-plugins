## nova-unity-bridge

> Session watcher that forwards OpenClaw assistant responses to Unity 3D visualization. Monitors session files and sends messages via HTTP bridge to Unity WebGL.


# Nova Unity Bridge

## Overview

Real-time session watcher that forwards OpenClaw conversations to Unity 3D visualization.

## Installation

```bash
# Clone
git clone https://github.com/zoltanferenczfi/nova-unity-bridge.git
cd nova-unity-bridge

# Setup (auto-detects everything)
./setup.sh
```

## What It Does

- Monitors `.jsonl` session files for new assistant messages
- Extracts message content (skips TTS, metadata)
- Queues messages with configurable delay
- Sends to Unity via HTTP POST to bridge endpoint

## Configuration

Environment variables (auto-detected if not set):

| Variable | Purpose |
|----------|---------|
| `OPENCLAW_DIR` | OpenClaw installation path |
| `SESSIONS_DIR` | Sessions directory path |
| `DEVICE_ID` | Device identifier for Unity |
| `UNITY_URL` | Unity bridge endpoint URL |
| `POLL_INTERVAL` | Poll interval (ms) |
| `MESSAGE_DELAY_MS` | Message delay (ms) |

## Architecture

```
┌──────────────┐     ┌─────────────────┐     ┌─────────────┐
│ OpenClaw     │────▶│ Session Watcher │────▶│ Unity       │
│ Session File │     │ (This Script)   │     │ WebGL       │
└──────────────┘     └─────────────────┘     └─────────────┘
```

## Technical Details

- Uses `fs.watch` pattern for file monitoring
- Tracks file position to avoid re-processing
- Deduplicates messages via ID tracking
- Implements queue with delay between sends
- Graceful error handling

## Files

- `session-watcher.js` - Main watcher implementation
- `setup.sh` - Auto-setup script
- `.env` - Environment configuration

## Requirements

- Node.js v18+
- PM2 (optional, for process management)
- OpenClaw installation

## License

MIT

---
> Source: [zoltanferenczfi/nova-unity-bridge](https://github.com/zoltanferenczfi/nova-unity-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
