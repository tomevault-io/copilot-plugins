## number-munchers

> This repository contains a JavaFX game and a local desktop automation service used for evaluation.

# Claude Workspace Notes

This repository contains a JavaFX game and a local desktop automation service used for evaluation.

## Key Paths

- Game root: `F:\data\workspaces\pairsys\number-munchers`
- Game source: `src/main/java/com/pairsys/numbermunchers/`
- Desktop agent: `codex-desktop-agent/`

## Required Behavior

Before attempting to use the desktop agent, check whether it is already running by requesting:

```powershell
Invoke-RestMethod http://127.0.0.1:8000/health
```

If the health check fails, start the agent yourself and then continue:

```powershell
cd F:\data\workspaces\pairsys\number-munchers\codex-desktop-agent
uv run python main.py
```

Do not stop to ask the user to start it if you can do so yourself.

## How To Evaluate The Game

- Prefer the JavaFX debug API on `http://127.0.0.1:8765` for structured game inspection.
- Use the desktop agent on `http://127.0.0.1:8000` for session launch, screenshots, window management, and fallback desktop automation.
- Prefer deterministic sessions by using a fixed seed.

## Desktop Agent Endpoints

- `GET /health`
- `GET /sessions`
- `POST /sessions/start`
- `GET /sessions/{session_id}/state`
- `POST /sessions/{session_id}/command`
- `POST /sessions/{session_id}/stop`

## Game Debug Endpoints

- `GET /health`
- `GET /state`
- `POST /command/reset`
- `POST /command/set-seed?seed=123`
- `POST /command/move?direction=left|right|up|down`
- `POST /command/munch`
- `POST /command/pause`
- `POST /command/resume`
- `POST /command/tick-enemies?count=1`

## Guidance

- Use debug-state reads for assertions whenever possible.
- Use screenshots for visual confirmation, not as the primary source of truth.
- Prefer the HTTP desktop agent workflow in this repo. Do not rely on MCP for `desktop` here.
- Keep workflow Windows-native and compatible with Gradle and `uv`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/amin3141) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
