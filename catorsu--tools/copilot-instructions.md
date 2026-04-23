## tools

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Environment Setup

The project requires Python 3.x and uses a virtual environment. Required packages are listed in requirements.txt:

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

## Project Architecture

This is a Python-based toolbox application with a plugin architecture:

- Core components provide base classes and plugin management
- Tools are implemented as plugins that inherit from base classes
- GUI is built using customtkinter library with a dark theme
- Tools are registered with the launcher at runtime

Key architectural patterns:
- Tools must implement both GUI and core functionality components
- Each tool lives in its own package under src/tools/
- Tool packages follow the pattern: __init__.py, gui.py, tool.py, plus tool-specific modules

## Running the Application

To run the application:

```bash
python main.py
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/catorsu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
