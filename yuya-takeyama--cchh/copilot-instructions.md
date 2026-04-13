## cchh

> This file provides comprehensive guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides comprehensive guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CCHH (Claude Code Hook Handlers) is a modular Python application that handles Claude Code hook events and dispatches them to various notification systems.

### Tech Stack
- **Language**: Python 3.13+
- **Package Manager**: uv
- **Task Runner**: taskipy
- **Formatter & Linter**: ruff
- **Testing**: pytest + pytest-cov
- **Type Checking**: mypy
- **CI/CD**: GitHub Actions

## Development Commands

### Essential Commands
```bash
# Install development tools (including uv)
aqua install

# Install dependencies
uv sync

# Run all tests with coverage
uv run task test

# Run linting
uv run task lint

# Format code
uv run task format

# Type checking
uv run task typecheck

# Run all checks (lint + test + typecheck)
uv run task all

# Clean build artifacts
uv run task clean
```

### Specific Test Commands
```bash
# Run specific test module
uv run pytest tests/slack/test_notifier.py -v

# Run specific test class
uv run pytest tests/core/test_dispatcher.py::TestEventDispatcher -v

# Run with coverage report
uv run pytest --cov=src --cov-report=term-missing
```

### Direct Commands (Alternative)
```bash
uv run pytest             # Run tests directly
uv run ruff check .       # Lint directly
uv run ruff format .      # Format directly
uv run mypy src          # Type check directly
```

## Project Structure

```
cchh/
├── pyproject.toml           # Python project config (uv + taskipy + ruff)
├── README.md               # User documentation
├── CLAUDE.md               # This file - Claude Code guidance
├── uv.lock                # Dependency lock file
├── all_hooks.py           # Main entry point
├── ruff_hook.py           # Ruff formatter hook
├── src/                   # Main package
│   ├── __init__.py
│   ├── core/               # Core functionality
│   │   ├── __init__.py
│   │   ├── base.py         # HookHandler interface
│   │   ├── dispatcher.py   # Event dispatcher
│   │   └── types.py        # Type definitions
│   ├── slack/              # Slack notification
│   │   ├── __init__.py
│   │   ├── notifier.py     # Main Slack notifier
│   │   ├── session_tracker.py # Session management
│   │   ├── event_formatter.py # Event message formatting
│   │   ├── command_formatter.py # Command formatting
│   │   └── config.py       # Slack configuration
│   ├── zunda/              # Zunda voice notifications
│   │   ├── __init__.py
│   │   ├── speaker.py      # Voice speaker
│   │   ├── prompt_formatter.py # Prompt formatting
│   │   ├── command_formatter.py # Command formatting (voice)
│   │   └── config.py       # Zunda configuration
│   ├── logger/             # Event logging
│   │   ├── __init__.py
│   │   ├── event_logger.py # JSONL logger
│   │   └── config.py       # Logger configuration
│   └── utils/              # Utilities
│       ├── __init__.py
│       ├── command_parser.py # Command parsing
│       ├── text_utils.py    # Text processing utilities
│       ├── io_helpers.py    # I/O helpers
│       ├── config.py        # Common configuration
│       └── logger.py        # Debug logger
├── tests/                  # Unit tests
│   ├── __init__.py
│   ├── conftest.py         # pytest configuration
│   ├── core/
│   ├── slack/
│   ├── zunda/
│   ├── logger/
│   └── integration/
├── event_logger.sh         # Event logging script (legacy)
├── test_problems.md       # Test issues report
├── LICENSE                # Project license
├── .gitignore             # Git ignore rules
├── .claude/               # Claude Code configuration
│   └── settings.local.json
├── aqua/                   # Tool management with aqua
│   ├── aqua.yaml
│   └── aqua-checksums.json
└── .github/
    └── workflows/
        └── test.yaml       # CI configuration
```

## High-Level Architecture

### Event Flow
1. **Entry Point**: `all_hooks.py` receives JSON events from Claude Code via stdin
2. **Event Dispatcher**: `src/core/dispatcher.py` routes events to enabled handlers
3. **Handlers**: Three main handlers process events in parallel:
   - **Slack Notifier**: Sends notifications to Slack with thread-based session tracking
   - **Zunda Speaker**: Provides voice feedback using Zundamon text-to-speech
   - **Event Logger**: Logs all events to JSONL format for analysis

### Key Design Patterns
- **Plugin Architecture**: Each handler implements the `HookHandler` interface from `src/core/base.py`
- **Dependency Injection**: Handlers are initialized conditionally based on environment variables
- **Session Management**: Slack handler maintains session state in `~/.cchh/slack_threads/` and `~/.cchh/sessions/`
- **Error Isolation**: Each handler's errors are caught independently to prevent cascading failures

### Core Features

#### Hook Event Types
- **PreToolUse**: Pre-tool execution processing
- **PostToolUse**: Post-tool execution processing
- **Notification**: Claude notifications
- **Stop**: Session termination
- **UserPromptSubmit**: User prompt submission
- **PreCompact**: Context overflow warning

#### Notification Systems
- **Slack notifications**: Commands, errors, session tracking
- **Voice feedback**: Audio guidance via Zundamon TTS
- **Event logging**: Structured logging in JSONL format

### Module Organization
- `src/core/`: Core types and event dispatching logic
- `src/slack/`: Slack integration with session tracking and message formatting
- `src/zunda/`: Voice notification system with command parsing
- `src/logger/`: JSONL event logging with rotation
- `src/utils/`: Shared utilities for I/O, parsing, and configuration

## Configuration

### Environment Variables

#### Slack Configuration
- `CCHH_SLACK_BOT_TOKEN`: Slack Bot Token (xoxb-...)
- `CCHH_SLACK_CHANNEL_ID`: Notification channel ID
- `CCHH_SLACK_NOTIFICATIONS_ENABLED`: Enable/disable Slack notifications (default: true)
- `CCHH_SLACK_SHOW_SESSION_START`: Show cwd on session start (default: true)
- `CCHH_SLACK_NOTIFY_ON_TOOL_USE`: Notify on tool usage (default: true)
- `CCHH_SLACK_NOTIFY_ON_STOP`: Notify on stop events (default: true)
- `CCHH_SLACK_COMMAND_MAX_LENGTH`: Max command display length (default: 200)

#### Zunda Voice Configuration
- `CCHH_ZUNDA_SPEAKER_ENABLED`: Enable/disable voice feedback (default: true)

#### Event Logging Configuration
- `CCHH_EVENT_LOGGING_ENABLED`: Enable/disable event logging (default: true)
- `CCHH_LOG_MAX_SIZE_MB`: Max log file size in MB (default: 100)
- `CCHH_LOG_ROTATION_COUNT`: Log rotation count (default: 5)

#### Other
- `CCHH_TEST_ENVIRONMENT`: Test environment flag (disables notifications during tests)
- `CCHH_CLAUDE_SESSION_ID`: Claude session ID

## Code Quality Configuration

### Ruff Settings (pyproject.toml)
- **Target**: Python 3.13
- **Line length**: 88 characters
- **Rules**: pycodestyle, pyflakes, isort, flake8-bugbear, pyupgrade
- **Format**: Double quotes, space indentation

### Test Configuration
- **Test path**: `tests`
- **Coverage**: term-missing reports
- **Branch coverage**: Enabled
- **Test files**: In `tests/` directory with `test_*.py` pattern
- **Coverage threshold**: Aim for >85%

### Testing Strategy
- Unit tests for each module
- Mock external dependencies (Slack API, filesystem)
- Test coverage >85%
- Type checking with mypy

## Claude Code Integration

### Hook Configuration Example

Add to Claude Code settings (`~/.claude/settings.json`):

```json
{
  "permissions": {
    "defaultMode": "acceptEdits"
  },
  "hooks": {
    "PreToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "cd /path/to/cchh && uv run python all_hooks.py"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "cd /path/to/cchh && uv run python all_hooks.py"
          }
        ]
      },
      {
        "hooks": [
          {
            "type": "command",
            "command": "cd /path/to/cchh && uv run python ruff_hook.py"
          }
        ]
      }
    ],
    "Notification": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "cd /path/to/cchh && uv run python all_hooks.py"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "cd /path/to/cchh && uv run python all_hooks.py"
          }
        ]
      }
    ],
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "cd /path/to/cchh && uv run python all_hooks.py"
          }
        ]
      }
    ],
    "PreCompact": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "cd /path/to/cchh && uv run python all_hooks.py"
          }
        ]
      }
    ]
  }
}
```

**Important Notes:**
- Event names use PascalCase (PreToolUse, PostToolUse, etc.)
- Each event is an array containing hooks objects
- Replace `/path/to/cchh` with actual repository path
- PostToolUse runs both event notification and Ruff formatting

## Special Files

### Entry Points
- `all_hooks.py`: Main entry point handling all hook events
- `ruff_hook.py`: Standalone hook for PostToolUse that runs Ruff formatter on Python files

### Legacy Files
- `event_logger.sh`: Simple shell script for basic event logging (legacy)

## Debugging and Monitoring

### Log Files
- `~/.cchh/logs/events.jsonl`: All hook events in JSONL format
- `~/.cchh/errors.log`: Error logs
- `~/.cchh/slack_threads/`: Slack thread state files
- `~/.cchh/sessions/`: Session state files

### Common Issues
1. **Import errors**: Ensure PYTHONPATH includes project root
2. **Slack not working**: Check CCHH_SLACK_BOT_TOKEN and CCHH_SLACK_CHANNEL_ID
3. **Tests failing**: Run `aqua install` then `uv sync` to update dependencies

### Debug Mode
Set `CCHH_TEST_ENVIRONMENT=1` to disable external notifications during development.

## Development Guidelines

### Code Style
- Follow PEP 8 (enforced by ruff)
- Use type hints for all functions
- Write docstrings for public APIs
- Keep functions focused and testable

### Testing Guidelines
- Write tests for new features
- Maintain test coverage >85%
- Use descriptive test names
- Mock external dependencies

### Performance Considerations
- Async notifications to avoid blocking CLI
- Efficient session state caching
- Log rotation for high-volume usage

## Claude Code Hook Event Format

### Input Format
Claude Code sends hook events in **flat JSON format** to stdin. The exact schema for each event type is defined in `src/core/types.py` using TypedDict.

### Event Type Definitions
- `BaseHookInput`: Common fields for all events (hook_event_name, session_id, transcript_path, cwd)
- `PreToolUseInput`: Input for tool execution events
- `PostToolUseInput`: Tool execution results
- `NotificationInput`: Notification events (uses 'message' field)
- `UserPromptSubmitInput`: User prompt submissions
- `StopInput`: Session termination events
- `PreCompactInput`: Context overflow warnings

### Important Notes
- Events are sent as **flat JSON**, not nested (no `{"data": {...}}` wrapper)
- The `message` field in Notification events is internally mapped to `notification` for compatibility
- Event logger outputs format: `{"time":"...", "raw_input":{...}}`
- See `src/core/types.py` for complete TypedDict schemas and field descriptions
- Reference: https://docs.anthropic.com/en/docs/claude-code/hooks#hook-input

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yuya-takeyama)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yuya-takeyama)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
