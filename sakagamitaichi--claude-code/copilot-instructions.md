## claude-code

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Python project named "claude-code" - a minimal application currently in initial development stages.

## Project Structure

- [main.py](main.py) - Entry point with a simple main function
- [pyproject.toml](pyproject.toml) - Python project configuration using modern packaging standards
- Requires Python >=3.10

## Development Commands

### Running the Application
```bash
python main.py
```

### Package Management
This project uses `pyproject.toml` for dependency management. When dependencies are added in the future, they should be listed in the `dependencies` array under `[project]`.

## Environment Setup

The project uses `.venv` as the virtual environment directory (gitignored). To set up:
```bash
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SakagamiTaichi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SakagamiTaichi)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
