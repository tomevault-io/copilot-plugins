## enikk

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Run Development Server
```bash
uv run enikk
```
This starts the Enikk daemon with all services (FastAPI server, IM bridge, etc.).

### Testing
```bash
# Run all tests
uv run pytest tests/ -v

# Run specific test file
uv run pytest tests/test_im_bridge.py -v

# Run with coverage
uv run pytest tests/ --cov=enikk
```

### Linting & Type Checking
```bash
# Run linter
uv run ruff check .

# Run type checker
uv run mypy enikk/ tests/
```

### Building
```bash
# Debug build (with console window)
.\build.bat

# Release build (no console, requires UAC)
.\build.bat --release
```
Output: `dist/enikk/enikk.exe` (release) or `dist/enikk-debug/enikk-debug.exe` (debug)

## Architecture

### Core Components

**Entry Point** (`__main__.py`)
- Initializes all services: FastAPI server, IM bridge, UI parser
- Handles graceful shutdown on interrupt

**Eternity** (`eternity.py`)
- Session manager for AI agent conversations
- Uses hermes-agent's SessionDB for persistence
- Manages multiple concurrent agent sessions

**Controller** (`controller.py`)
- Main agent logic using hermes-agent's AIAgent
- Coordinates UI parser, game capture, and input services
- Provides tool implementations: analyze, click, wait_for, etc.

**UI Parser** (`ui_parser.py`)
- Two-stage detection: YOLO (icons) + RapidOCR (text)
- Returns unified UI elements with normalized coordinates [0, 1000]
- Handles overlapping detections with priority rules

**Game Control** (`game/` module)
- `capture.py`: Screenshot via win32gui (foreground window)
- `input.py`: Mouse/keyboard via win32api (background input)
- `window.py`: Window discovery and focus management
- `process.py`: Process monitoring and termination

**Server** (`server.py`)
- FastAPI HTTP server (port 9901)
- REST API for session management, agent control
- Static file serving from `enikk/static/`

**IM Bridge** (`im_bridge.py`)
- Connects to IM platforms (QQ via NapCat)
- Routes messages to Eternity sessions
- Streams agent responses back to IM

### Data Flow

```
IM Message → IMBridge → Eternity (creates session)
                              ↓
                      Controller (AIAgent)
                              ↓
                    ┌───── Tool Calls ─────┐
                    ↓                      ↓
              UI Parser            Game Control
           (YOLO + OCR)      (capture, input)
                    ↓                      ↓
                    └──── Results ─────────┘
                              ↓
                      IM Response (streaming)
```

### Configuration

- Config stored at `%USERPROFILE%/.enikk/config.yaml`
- Model weights at `%USERPROFILE%/.enikk/weights/`
- Session data at `%USERPROFILE%/.enikk/sessions/`

### Key Dependencies

- **hermes-agent**: AI agent framework, session management
- **FastAPI + Uvicorn**: HTTP server
- **RapidOCR**: Chinese/English text recognition
- **onnxruntime**: YOLO icon detection
- **pywebview**: Native window for web UI
- **pywin32**: Windows API for window/input control

## Conventions

- **No Co-Authored-By in commits.** Do not include `Co-Authored-By` trailer in git commit messages.

---
> Source: [gtt116/enikk](https://github.com/gtt116/enikk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
