## xiaozhi-pi-mochi

> - **`.pi/extensions/xiaozhi-bot/`** — pi-agent extension implementing the xiaozhi-esp32 voice chatbot backend. This is the repo's own code.

# talktoy — workspace for xiaozhi-esp32 backend

## Structure

- **`.pi/extensions/xiaozhi-bot/`** — pi-agent extension implementing the xiaozhi-esp32 voice chatbot backend. This is the repo's own code.
- **`pi/`** — git submodule for [pi-agent](https://github.com/earendil-works/pi-mono) (TypeScript monorepo, npm workspace)
- **`xiaozhi-esp32/`** — git submodule for [xiaozhi-esp32](https://github.com/78/xiaozhi-esp32) (ESP-IDF C firmware)

Both submodules are tracked as gitlinks (160000 mode). Initialize after clone:

```powershell
git submodule update --init --recursive
```

## Entrypoint

`.pi/extensions/xiaozhi-bot/index.ts` — WebSocket server bridging ESP32 devices to an ASR+LLM+TTS pipeline using pi-agent's AI stack.

## Dependencies

| Dep | Version | Use |
|-----|---------|-----|
| Node.js | >= 18 | pi-agent + extension runtime |
| Python | >= 3.10 | ASR (faster-whisper), TTS (edge-tts) |
| ffmpeg | any | audio transcoding |
| uvx or Python | any | running edge-tts |

Python packages: `pip install edge-tts faster-whisper av`

## Setup

```powershell
# 1. Install the extension into pi-agent
pi install .pi/extensions/xiaozhi-bot

# 2. Install extension Node deps
cd .pi/extensions/xiaozhi-bot; npm install
```

## Run

**Auto-start** (recommended):

```powershell
$env:XIAOZHI_BOT=1; pi
```

**Manual** inside pi:
```
> /xiaozhi start
```

Port defaults to 8989. Override with `$env:XIAOZHI_PORT` or `/xiaozhi port <n>`, then restart.

ESP32 devices connect to `ws://<host>:<port>` with protocol v1/v3 auto-detected.

## Commands (inside pi interactive)

| Command | Description |
|---------|-------------|
| `/xiaozhi start [port]` | Start WebSocket server |
| `/xiaozhi stop` | Stop server |
| `/xiaozhi status` | Server + connected devices |
| `/xiaozhi session <id>` | Inspect device session |
| `/xiaozhi sysprompt <id> <prompt>` | Set per-device system prompt |

## Test

```powershell
# Start server first in one terminal
$env:XIAOZHI_BOT=1; $env:XIAOZHI_PORT=19989; pi

# In another terminal:
node .pi/extensions/xiaozhi-bot/test.mjs      # basic protocol handshake
node .pi/extensions/xiaozhi-bot/full-test.mjs  # full ASR+LLM+TTS pipeline end-to-end
```

Requires Python + ffmpeg + GPU or CPU whisper model.

## Env variables

| Var | Default | Description |
|-----|---------|-------------|
| `XIAOZHI_BOT` | - | Set to `1` to auto-start server |
| `XIAOZHI_PORT` | `8989` | WebSocket server port |

## VSCode

`.vscode/settings.json` points to an IDF Python install for ESP32 development.

## pi submodule

See `pi/AGENTS.md` and `pi/CONTRIBUTING.md` for upstream coding agent development rules, commit conventions, and release process.

---
> Source: [hoorayman/xiaozhi-pi-mochi](https://github.com/hoorayman/xiaozhi-pi-mochi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
