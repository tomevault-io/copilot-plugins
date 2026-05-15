## project-structure

> This is an [Arcade.dev](https://arcade.dev) toolkit demonstrating professional Python development practices for AI tool orchestration. The project structure is:


# Arcade.dev Found Audio Toolkit - Project Structure

This is an [Arcade.dev](https://arcade.dev) toolkit demonstrating professional Python development practices for AI tool orchestration. The project structure is:

## Core Architecture
- **Main Package**: [foundaudio/foundaudio/](mdc:foundaudio/foundaudio/) - Core toolkit package
- **Tools**: [foundaudio/foundaudio/tools/](mdc:foundaudio/foundaudio/tools/) - Arcade tools implementation
- **Tests**: [foundaudio/tests/](mdc:foundaudio/tests/) - Comprehensive test suite with mocking
- **Evals**: [foundaudio/evals/](mdc:foundaudio/evals/) - Evaluation suite for tool performance
- **Configuration**: [foundaudio/pyproject.toml](mdc:foundaudio/pyproject.toml) - Project metadata and dependencies

## Key Files
- **Entry Point**: [foundaudio/foundaudio/__init__.py](mdc:foundaudio/foundaudio/__init__.py) - Package initialization and tool exports
- **Tool Registration**: [foundaudio/foundaudio/tools/__init__.py](mdc:foundaudio/foundaudio/tools/__init__.py) - Tool catalog registration
- **Main Tool**: [foundaudio/foundaudio/tools/get_audio_list.py](mdc:foundaudio/foundaudio/tools/get_audio_list.py) - Primary audio search functionality
- **Development**: [foundaudio/Makefile](mdc:foundaudio/Makefile) - Build, test, and development commands

## Technology Stack
- **Package Manager**: uv (modern Python package management)
- **Testing**: pytest with async support and comprehensive mocking
- **Database**: Supabase Python client for Found Audio API
- **Framework**: Arcade TDK (Toolkit Development Kit)
- **Quality**: Trunk metalinting, mypy type checking, pre-commit hooks

## Development Philosophy
This project emphasizes **Foundation First** - establishing solid development practices before adding features, with comprehensive testing, proper error handling, and professional documentation standards.

---
> Source: [rsmets/arcade-foundaudio-toolkit](https://github.com/rsmets/arcade-foundaudio-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
