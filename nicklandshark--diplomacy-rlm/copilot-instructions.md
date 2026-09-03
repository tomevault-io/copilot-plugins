## diplomacy-rlm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Diplomacy-RLM is a Python harness that runs autonomous Diplomacy game powers using the RLM (Recursive Language Model) framework. Each power is controlled by LLM agents that strategize, negotiate via messages, and submit orders through a sandboxed Python REPL. A Next.js web viewer provides live game observation via SSE.

## Build & Run Commands

```bash
# Install (editable, with web deps)
uv pip install -e ".[web]"

# Install frontend
cd web && bun install && cd ..

# Run tests
uv run pytest

# Run a single test file
uv run pytest tests/test_orchestrator.py

# Run a single test by name
uv run pytest tests/test_orchestrator.py -k "test_name"

# Run game (minimal two-power example)
uv run diplomacy-rlm \
  --powers FRANCE,GERMANY \
  --backend anthropic \
  --backend-arg-for anthropic.model_name=claude-opus-4-6 \
  --max-year 1903

# Run with web viewer
uv run diplomacy-rlm \
  --powers FRANCE,GERMANY \
  --backend anthropic \
  --backend-arg-for anthropic.model_name=claude-opus-4-6 \
  --serve-web --web-port 3000 --log-events
```

## Architecture

### Core Game Loop (Orchestrator)

`orchestrator.py` drives the game through Diplomacy phases. Each **Movement phase** has three steps run concurrently across powers:

1. **STRATEGIZE** — Each power's `StrategistAgent` analyzes the board and optionally emits `SPAWN_CONVERSATION({"TARGET": "objective"})` to request negotiations.
2. **CONVERSE** — Ephemeral `ConversationAgent`s are created per power to exchange messages over multiple rounds. Messages flow through `MessageRouter` (queue → flush → game).
3. **DECIDE** — Strategists receive conversation summaries, choose final orders via `submit_orders([...])`, update memory, then `FINAL(done)`.

**Retreat (R)** and **Adjustment (A)** phases skip STRATEGIZE/CONVERSE and run DECIDE only.

All agents run concurrently in daemon thread pools (`_BestEffortDaemonPool`) with per-step timeouts controlled by `PhaseTimer`. Timed-out agents get default orders (hold/disband/waive).

### Agent System

- **`StrategistAgent`** (`agents/strategist.py`) — Persistent per-power agent across all phases. Wraps an RLM instance with a sandboxed Python REPL. Injected globals: `game_view`, `memory_path`, `time_remaining()`, `submit_orders()`.
- **`ConversationAgent`** (`agents/conversation.py`) — Ephemeral per-phase diplomat. Injected globals: `game_view` (FilteredGameView with target-scoped messages), `memory_snapshot`, `send_message()`, `objectives`.
- Both agents communicate with the game engine through sentinel text patterns (`FINAL(...)`, `SPAWN_CONVERSATION({...})`) parsed by `sentinels.py`.

### RLM Runtime (`rlm_runtime.py`)

Wraps the external `rlm` package. Falls back to `FallbackRLM` (a scripted stub) when the real package is unavailable—this is how tests run without API keys. The `install_env_hook()` function intercepts RLM's environment lifecycle to inject/extract agent globals at the right moment.

### Two Execution Modes

- **Local** (`--sandbox local`): Agents run in-process with a persistent Python environment. `GameView` wraps the live game object directly. Sandbox guards (`repl_sandbox.py`) block dangerous modules and restrict file access.
- **Modal** (`--sandbox modal`): Agents run in remote Modal sandboxes. `modal_bridge.py` serializes game state as JSON snapshots, sends setup code with `_SnapshotGameView` (a read-only dict-backed replica), and exports results (orders, memory, messages) back.

### Key Data Flow

```
GameView/FilteredGameView (read-only game access)
       ↓
StrategistAgent / ConversationAgent (RLM completions in sandboxed REPL)
       ↓
MessageRouter (thread-safe queue/flush for diplomatic messages)
       ↓
Orchestrator (applies orders to Game, processes phase, saves snapshots)
```

### Observability (`observability/`)

- `EventEmitter` base class with implementations: `NoopEmitter`, `RecorderEmitter` (tests), `ConsoleEventLogger`, `BufferedEventBus` (SSE).
- `TeeEmitter` multiplexes to multiple emitters (e.g., console + SSE bus).
- `api.py` — FastAPI SSE sidecar (`/events/stream`, `/health`, `/state`).
- Events follow `ObservableEvent` schema with `event_type`, `priority`, `phase`, `step`, `power`, `payload`.

### Web Viewer (`web/`)

Next.js 16 app (React 19, Tailwind CSS 4, SWR, bun). Reads game data from:
- File-based API routes (`/api/games/[gameId]/...`) that read snapshot JSON from the `--game-dir`.
- Live SSE stream from the FastAPI sidecar (env var `LIVE_API_URL`).

### Vendored Diplomacy Engine (`_vendor/diplomacy/`)

Vendored fork of the `diplomacy` Python package providing `Game`, `Map`, `Power`, `Message`, and order adjudication. The `Game` object is the single source of truth for board state, and agents interact with it only through `GameView` wrappers.

## Testing

Tests use `ScriptedRLM` (defined in `tests/conftest.py`), a deterministic double that replaces real RLM instances via `monkeypatch`. Use the `patched_agent_rlm` fixture to swap in `ScriptedRLM` for both strategist and conversation agent modules. Queue responses with `scripted_rlm.queue_response("...")`.

The `RecorderEmitter` captures all emitted events for assertion in tests.

## Key Conventions

- Power names are always uppercase strings: `AUSTRIA`, `ENGLAND`, `FRANCE`, `GERMANY`, `ITALY`, `RUSSIA`, `TURKEY`.
- All power name inputs are normalized via `normalize_power_name()` / `normalize_powers()`.
- The `GameConfig` dataclass centralizes all configuration; per-power backend/model overrides are resolved via `backend_for(power)` and `backend_kwargs_for(power)`.
- Agents use step/round tokens for timeout guard: `activate_step_token()` / `invalidate_step_token()` prevent stale completions from affecting state after timeouts.
- `submit_orders()` is once-only per phase — the first call is final.
- Memory is a per-power markdown file (`POWER_memory.md`) in the game directory, read/written by agents through the sandboxed `open()`.

---
> Source: [nicklandshark/diplomacy-rlm](https://github.com/nicklandshark/diplomacy-rlm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
