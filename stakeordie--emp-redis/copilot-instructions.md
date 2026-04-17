## emp-redis

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Run Commands
- Hub service: `cd hub && python main.py`
- Worker service: `cd worker && python main.py`
- Docker deployment: `cd apps/mock-env && docker-compose up`
- Test logger: `python test_logger.py`
- Worker with simulation connector: Set `CONNECTORS=simulation` in .env

## Linting and Type Checking
- Use mypy for Python type checking: `mypy [file_or_directory]`
- Python version: 3.10+

## Code Style Guidelines
- Use Python type annotations for all function parameters and return types
- Follow interface-based design patterns (see interfaces/ directory)
- Implement all abstract methods from interfaces
- Use Redis URL format for connection configuration: `redis://[[username]:[password]@][host][:port][/database]`
- Core modules in core/ are used by all components - be cautious when modifying
- Use modular connector-based architecture for workers
- Ensure compatibility between Hub and Worker components
- Use proper naming conventions: snake_case for Python, camelCase for TypeScript

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stakeordie) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
