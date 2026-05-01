## chat-stick

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A handheld voice assistant built on an M5StickS3 (ESP32-S3). Hold a button, talk, release to hear the AI respond. Audio streams over WiFi/WebSocket to a Cloudflare Worker, which relays it to Google's Gemini 3.1 Live API for speech-to-speech AI.

```
M5StickS3 ──WebSocket──▶ Cloudflare Worker (Durable Object) ──WebSocket──▶ Gemini Live API
  mic/speaker               relay + tool handling                           speech-to-speech AI
```

## Repository Structure

Two independent codebases in one repo:

- **`firmware/`** — PlatformIO/Arduino C++ project targeting M5StickS3 (ESP32-S3)
- **`server/`** — Cloudflare Worker (TypeScript) with Durable Objects, D1, Vectorize, Workers AI

## Build & Run Commands

### Server (Cloudflare Worker)

```bash
cd server
npm install
npm run dev          # wrangler dev (local)
npm run deploy       # wrangler deploy (production)

# D1 migrations
wrangler d1 migrations apply --local

# Docs indexing
DOCS_PATH=/path/to/docs npx tsx scripts/build-docs-index.ts
wrangler dev scripts/index-docs-worker.ts --port 8799
curl http://localhost:8799/index
```

### Firmware (PlatformIO)

```bash
cd firmware
pio run -t upload          # build & flash
pio device monitor         # serial monitor (115200 baud)
python monitor.py          # alternative serial monitor
```

Serial port is configured in `platformio.ini` (`upload_port`/`monitor_port`). Update these when switching USB ports.

## Architecture Details

### Server

- **Entry point**: `server/src/index.ts` — HTTP router handling `/ws`, `/health`, `/history/:deviceId`, `/session/:chatId`, `/firmware/check`, `/firmware/download`, and admin endpoints
- **`LiveSession` Durable Object** (`server/src/live-session.ts`) — one instance per device. Manages dual WebSocket connections (device ↔ Gemini), routes tool calls, tracks transcriptions, persists conversation history to D1
- **Tool call routing**: Tools like `search_docs`, `web_fetch`, `new_conversation` are handled server-side; device-control tools (`set_brightness`, `set_volume`, `show_text`, `play_sound`, `play_melody`, `power_off`, `get_device_status`) are forwarded to the device via WebSocket and the response is relayed back to Gemini
- **Docs search** (`server/src/docs-search.ts`) — keyword search (in-memory JSON index) with vector search fallback (Cloudflare Vectorize + Workers AI embeddings)
- **D1 schema** (`server/schema.sql`) — `conversations`, `message_log`, `tool_log` tables

### Firmware

- **`main.cpp`** — thin shell delegating to `AppController`
- **`AppController`** (`app/`) — central coordinator. Owns all services, manages state machine (`AppState`: Connecting → Ready → Recording → Thinking → Playing), handles button input, menu navigation, and display updates
- **Services** (`services/`):
  - `LiveSessionService` — WebSocket client to the server; handles connection, audio streaming, tool call dispatch, session restore, firmware update checks
  - `AudioService` — mic capture (16kHz) and speaker playback (24kHz PCM)
  - `WiFiService` — multi-network connection with saved credential support
  - `SettingsStore` — NVS persistence for brightness, volume, chat_id, WiFi credentials
- **`ButtonStateMachine`** (`input/`) — debounced press/long-press/release detection for A (push-to-talk) and B (menu) buttons
- **`PowerManager`** (`power/`) — idle timeout cascade: dim → screen off → light sleep → power off
- **`TextDisplay`** (`ui/`) — 135×240 LCD rendering with header/body/footer layout, menus, and state-based coloring
- **`Config.h`** — all hardware constants, server endpoints, audio rates, pin assignments, power timeouts

### Key Data Flows

1. **Voice exchange**: Button A press → mic capture at 16kHz → PCM chunks over WebSocket → server base64-encodes → Gemini `realtimeInput` → Gemini responds with audio parts → server decodes base64 → raw PCM binary frames back to device → speaker playback at 24kHz
2. **Tool calls**: Gemini emits `toolCall` → server handles server-side tools directly, forwards device-side tools as JSON → device executes and sends `tool_response` → server relays `toolResponse` to Gemini
3. **Session restore**: On boot, device sends saved `chat_id` → server fetches `last_message` from D1 → device displays previous conversation context

## Credentials

All secrets are gitignored. Templates exist at:
- `server/.dev.vars.example` → `server/.dev.vars` (GEMINI_API_KEY, HISTORY_API_TOKEN)
- `firmware/src/credentials.h.example` → `firmware/src/credentials.h` (WiFi networks)

## Audio Format

- Input (mic → Gemini): 16-bit PCM, 16kHz, mono
- Output (Gemini → speaker): 16-bit PCM, 24kHz, mono
- Short/silent clips are detected and ignored (MIN_TURN_BYTES, SILENCE_AVG_ABS_THRESHOLD)

---
> Source: [steveruizok/chat-stick](https://github.com/steveruizok/chat-stick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
