## prototype-termai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

prototype-termai is a Terminal AI Assistant that provides real-time AI-powered assistance for terminal operations using local LLM models through Ollama. The system watches terminal activity and offers contextual help, error solutions, and command suggestions in a non-intrusive sidebar.

**Current Status**: ~85% complete - all core features are functional and the application is ready for use.

## Key Architecture Components

### Core System Design
- **PTY-based Terminal Emulation**: Uses Python's `pty` module to create a fully functional terminal wrapper that captures all input/output while maintaining complete shell compatibility
- **Async Event-Driven Architecture**: Built on asyncio with an event bus system to ensure terminal operations are never blocked by AI processing
- **Smart Context Management**: Implements relevance scoring and context compression to work within Ollama's token limits (4K-8K tokens)
- **Real-time UI Updates**: Textual framework with separate terminal panel (65%) and AI sidebar (35%) that updates asynchronously

### Module Structure
```
src/termai/
├── terminal/    - Terminal emulation (TerminalEmulator, OutputBuffer, CommandHistory, TerminalManager)
├── ui/          - TUI components (TerminalAIApp, TerminalWidget, AISidebar)
├── ai/          - AI integration (8 modules: OllamaClient, ContextManager, RealtimeAnalyzer, etc.)
├── config/      - Configuration management (partial implementation)
└── utils/       - Core utilities (logger - partial implementation)

tests/           - Test files (terminal, ui, ollama, integration)
docs/            - Documentation including plan/ and status.md
examples/        - Usage examples (basic_usage.py)
```

## Development Commands

### Initial Setup
```bash
# Install uv (if not already installed)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install dependencies and create virtual environment
uv sync

# Install dev dependencies
uv sync --all-extras

# Install Ollama and models
ollama serve  # Run in separate terminal
ollama pull codellama:7b

# Set up pre-commit hooks
uv run pre-commit install

# Project structure already created during refactoring
```

### Running Tests
```bash
# Individual test modules
uv run python tests/test_terminal.py    # Terminal emulation tests
uv run python tests/test_ui.py          # UI component tests
uv run python tests/test_ollama.py      # AI integration tests
uv run python tests/test_integration.py # Integration tests

# Pytest (all tests)
uv run pytest tests/

# Run specific test
uv run pytest tests/test_terminal.py::test_function_name -v

# Code quality checks
uv run pre-commit run --all-files
uv run ruff check .
uv run black --check .
uv run mypy .
```

### Running the Application
```bash
# Main application (recommended)
uv run termai
uv run python -m termai
uv run python src/termai/main.py

# With debugging
uv run python src/termai/main.py --debug --log-level=DEBUG

# Performance profiling
uv run python -m cProfile -o profile.stats src/termai/main.py

# Example usage
uv run python examples/basic_usage.py
```

## Critical Implementation Details

### Core Architecture Flow
1. **TerminalManager** orchestrates all terminal operations and AI integration
2. **TerminalEmulator** uses PTY for full shell compatibility
3. **OutputBuffer** captures and processes terminal output with ANSI handling
4. **ContextManager** processes commands and triggers AI analysis requests
5. **RealtimeAnalyzer** manages async AI processing with caching and throttling
6. **AISidebar** displays AI responses in real-time without blocking terminal operations

### Terminal Integration
- PTY-based emulation ensures 100% shell compatibility
- Command detection via prompt pattern matching (bash/zsh/fish support)
- Real-time output streaming with circular buffer (1000 lines default)
- Command history with execution time and exit code tracking
- AI analysis hooks in `TerminalManager._end_command()` method

### AI System Architecture
The AI system consists of 8 modules:
- **ollama_client.py**: HTTP client for Ollama server
- **context_manager.py**: Orchestrates AI analysis requests
- **context.py**: Sliding window context management with relevance scoring
- **triggers.py**: Pattern matching for error/dangerous command detection
- **realtime_analyzer.py**: Async processing with caching and throttling
- **prompts.py**: Template system for different analysis types
- **filters.py**: Sensitive information filtering
- **__init__.py**: Package initialization

### Event-Driven Processing
- Command completion triggers context analysis
- Relevance scoring (0.3-0.9) determines AI engagement
- Priority queuing prevents LLM overload
- Response caching (30-40% hit rate) improves performance

## Environment Configuration

Create `.env` file with:
```
OLLAMA_HOST=http://localhost:11434
OLLAMA_MODEL=llama3.2:1b
OLLAMA_TIMEOUT=30
APP_LOG_LEVEL=INFO
AI_MAX_CONTEXT_LENGTH=20
AI_RESPONSE_MAX_TOKENS=500
```

**Note**: Current implementation primarily uses `llama3.2:1b` model for faster responses.

## Key Development Patterns

### Adding New AI Triggers
1. Add trigger to `ai/triggers.py` in `_init_default_triggers()`
2. Set priority (1-10, higher = more important)
3. Test with `tests/test_ollama.py`

### Terminal Integration
- AI hooks are in `terminal/manager.py` at `_end_command()` method
- Command context flows: command → directory → exit_code → output → error → duration
- Integration point: `TerminalManager.ai_analyzer` handles analysis requests

### UI State Management
- `ui/app.py`: Main application with 65%/35% layout
- `ui/terminal_widget.py`: Terminal display (test mode available)
- `ui/ai_sidebar.py`: AI responses with real-time updates
- Keyboard shortcuts: Ctrl+L (clear), Ctrl+A (AI toggle), F1 (help)

### AI Response Flow
1. Command completion triggers `ContextManager.process_command()`
2. Triggers are evaluated and analysis requests queued
3. `RealtimeAnalyzer` processes requests asynchronously
4. Responses are cached and displayed in `AISidebar`
5. UI updates happen via callback registration

## Implementation Status

Checkpoints completed:
1. ✅ **Checkpoint 1**: Terminal wrapper (100% complete)
2. ✅ **Checkpoint 2**: TUI framework (100% complete)
3. ✅ **Checkpoint 3**: Ollama AI integration (100% complete)
4. 🔄 **Checkpoint 4**: Full integration (~85% complete)

**Remaining work**: Performance optimization, settings management, integration tests.

See `docs/status.md` for detailed current state and `docs/plan/checkpoints/` for validation details.

## Important Notes

### Prerequisites
- Python 3.12+ (specified in pyproject.toml)
- Ollama server running (`ollama serve`)
- At least one model installed (`ollama pull llama3.2:1b`)
- uv package manager for dependency management

### Performance Characteristics
- Startup time: ~2-3 seconds (AI system initialization)
- Command response: < 50ms (terminal operations)
- AI analysis: 1-2 seconds average
- Memory usage: ~150-200MB
- Supports Korean language in UI

### Troubleshooting
See `docs/plan/troubleshooting.md` for common issues including:
- Ollama connection failures
- Terminal rendering problems
- Korean text encoding issues
- Performance optimization tips

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cagojeiger) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
