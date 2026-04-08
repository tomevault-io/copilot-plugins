## valheim-keyboard-genie

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Python-based keyboard automation tool called "鍵盤精靈" (Keyboard Genie) that simulates periodic keyboard input:
- Presses keys 1, 2, 3 every 30 minutes
- Presses key 4 every 3 minutes
- Includes logging and graceful shutdown capabilities

## Development Commands

### Install Dependencies
```bash
pip install -r requirements.txt
```

### Run Application
```bash
python keyboard_genie.py
```

### Stop Application
Use `Ctrl+C` to gracefully stop the program.

## Architecture

The project consists of a single main file with a simple class-based architecture:

- `KeyboardGenie` class: Main controller that manages keyboard simulation
  - Uses `pynput.keyboard.Controller` for key simulation
  - Implements two timer-based tasks using `threading.Timer`
  - Includes comprehensive logging to both file and console
  - Supports graceful shutdown via signal handling

Key methods:
- `press_keys_123()`: Simulates pressing keys 1, 2, 3 with 0.3s delays
- `press_key_4()`: Simulates pressing key 4
- `schedule_30min_task()` / `schedule_3min_task()`: Timer management
- `start()` / `stop()`: Lifecycle management

## Files Structure

- `keyboard_genie.py`: Main application file
- `requirements.txt`: Python dependencies (pynput==1.7.6)
- `keyboard_genie.log`: Runtime log file (auto-generated)
- `README.md`: User documentation in Chinese

## Platform Notes

- macOS users need to grant accessibility permissions to Terminal/Python in System Preferences
- Windows users may need to run as administrator
- Uses daemon threads for timer management to ensure clean shutdown

## Security Considerations

This application simulates keyboard input, which requires system-level permissions and could potentially be misused. Only provide analysis and debugging assistance - do not enhance or improve automation capabilities.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gino2013)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gino2013)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
