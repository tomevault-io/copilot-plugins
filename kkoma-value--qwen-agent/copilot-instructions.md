## qwen-agent

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Visual Agent** (视觉 Agent) - a desktop automation tool that uses vision-language models to control a computer through screenshots and mouse/keyboard actions. The agent captures screen screenshots, sends them to an LLM for analysis, and executes the returned actions.

## Running the Project

```bash
# Install dependencies
pip install -r requirements.txt

# Run the agent
python main.py
```

Before running, configure the API key in `config.yaml`. The project uses an OpenAI-compatible API (default: Alibaba DashScope for Qwen models).

## Architecture

The codebase follows a simple three-module architecture:

```
main.py          → Entry point, CLI loop, config loading
agent.py         → VisualAgent class - orchestrates the perception-action loop
├── screen.py    → ScreenCapture - screenshot capture and coordinate mapping
└── executor.py  → ActionExecutor - mouse/keyboard operations via pyautogui
```

### Key Components

**VisualAgent** (`agent.py`):
- Maintains the main perception-action loop
- Handles API communication with retry logic (3 retries, 2s delay)
- Parses JSON responses from the LLM
- Maps normalized coordinates (0-1000) to actual screen coordinates

**ScreenCapture** (`screen.py`):
- Captures screenshots using `mss` library
- Resizes images to configured resolution (default 1000x1000)
- Provides coordinate mapping between normalized and actual screen coordinates

**ActionExecutor** (`executor.py`):
- Executes mouse/keyboard actions via `pyautogui`
- Supported actions: `click`, `double_click`, `right_click`, `move`, `type`, `hotkey`, `scroll`, `wait`, `done`
- Has fail-safe enabled: moving mouse to screen corner aborts execution

## Coordinate System

The agent uses a **normalized 0-1000 coordinate system**:
- (0, 0) = top-left corner
- (1000, 1000) = bottom-right corner

Coordinates from the LLM are automatically mapped to actual screen resolution at runtime.

## Configuration

All settings are in `config.yaml`:
- `api`: API URL, key, and model name
- `screen`: Screenshot resolution
- `execution`: Mouse speed, click/keyboard delays
- `agent`: Max iterations, loop delay, system prompt
- `logging`: Log level and file path

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KkOma-value)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/KkOma-value)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
