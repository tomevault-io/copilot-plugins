## python-learning-assistant-cli

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Python-based interactive learning assistant that provides intelligent Q&A, code execution, syntax highlighting, and multi-turn conversations for Python learning. The application integrates with Moonshot AI API and includes advanced time management features for tracking learning sessions.

## Development Commands

### Environment Setup
```bash
# Install dependencies
pip install -r requirements.txt

# Set API key (required)
# Windows:
set MOONSHOT_API_KEY=your_api_key_here
# Linux/Mac:
export MOONSHOT_API_KEY=your_api_key_here
```

### Running the Application
```bash
# Primary method - automatic dependency check and startup
python run.py

# Alternative - direct execution
python src/main.py

# Package installation method
pip install -e .
python-learning-assistant
```

### Testing
```bash
# Run main test suite
python test.py

# Run specific test modules
python test_sessions.py
python test_sessions_standalone.py
python test_time_manager.py
```

### Development Installation
```bash
# Modern approach (recommended)
pip install -e .

# Traditional approach
python setup.py develop
```

## Architecture

### Core Components

**Main Application (`src/main.py`)**: The primary application featuring an interactive terminal interface with ANSI color support, command processing system, and integrated AI chat functionality.

**Time Manager (`src/time_manager.py`)**: Advanced session tracking with conversation history, learning metrics, and session persistence. Handles time-based analytics and learning progress tracking.

**Configuration System (`config/config.json`)**: Centralized configuration for AI model parameters, session settings, and application behavior.

### Key Architectural Patterns

- **Modular Design**: Core functionality separated into dedicated modules with optional time management integration
- **Graceful Degradation**: Time management features are optional - the application functions without them
- **Session Persistence**: Learning sessions are automatically saved to JSON files in the `sessions/` directory
- **Configuration-Driven**: All major behaviors controlled through `config/config.json`

### Application Flow

1. **Startup**: `run.py` performs dependency checks, validates project structure, and verifies API configuration
2. **Initialization**: Main application loads configuration, initializes time manager (if available), and sets up the interactive terminal
3. **Command Processing**: Interactive loop handles user commands (`/help`, `/quit`, `/save`, `/run`, etc.) and AI chat
4. **Session Management**: Conversations and learning metrics are tracked and persisted automatically

## Configuration

### Required Environment Variables
- `MOONSHOT_API_KEY`: Moonshot AI API key (required for AI functionality)

### Key Configuration Options (`config/config.json`)
- `model`: AI model name (default: "kimi-k2-0711-preview")
- `temperature`: Response randomness (0.0-2.0, default: 0.3)
- `max_tokens`: Maximum response length (default: 2048)
- `max_history`: Conversation history limit (default: 50)
- `code_timeout`: Code execution timeout in seconds (default: 10)
- `auto_save_sessions`: Automatic session saving (default: true)

## File Structure Notes

- **`src/`**: Core application code
- **`config/`**: Configuration files
- **`sessions/`**: Auto-generated session data (JSON format)
- **`examples/`**: Python learning examples and demonstrations
- **`docs/`**: Comprehensive documentation including usage guides and configuration details

## Development Notes

- Application supports both relative and absolute imports for the time manager module
- Terminal interface includes extensive Unicode/encoding fallback for cross-platform compatibility
- Code execution is sandboxed with configurable timeout limits
- Session data includes conversation history, timestamps, and learning analytics

---
> Source: [sheacoding/python_learning_assistant_cli](https://github.com/sheacoding/python_learning_assistant_cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
