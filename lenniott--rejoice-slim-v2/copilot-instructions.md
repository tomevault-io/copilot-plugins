## rejoice-slim-v2

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Rejoice Slim v2** - Local-first voice transcription tool. Terminal-only, privacy-first, zero data loss.

- **Name Meaning:** Rejoice = {record, jot, voice}, Slim = {super lightweight, no UI, no cloud, no crazy}
- **Core Philosophy:** Data integrity above all, simplicity over features, local-first/local-only, transparency over magic, boring reliability over clever features
- **Status:** In active development (Phase 1 complete, Phase 2 in progress)
- **License:** MIT

## Development Commands

### Environment Setup
```bash
# Activate virtual environment
source venv/bin/activate

# Install dependencies (development)
pip install -e ".[dev]"

# Install pre-commit hooks
pre-commit install
```

### Testing Commands
```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=src/rejoice --cov-report=html --cov-report=term

# Run specific test types
pytest tests/unit           # Unit tests only
pytest tests/integration    # Integration tests only
pytest tests/e2e            # End-to-end tests only

# Run single test
pytest tests/unit/test_config.py::test_load_default_config -v

# Run tests in watch mode (if pytest-watch installed)
ptw tests/unit
```

### Code Quality
```bash
# Format code
black src tests

# Lint
flake8 src tests

# Type check
mypy src

# Run all pre-commit checks
pre-commit run --all-files
```

### Running the CLI
```bash
# Using the installed command (after installation)
rec --help
rec --version
rec config show

# During development (from project root)
python -m rejoice --help
python -m rejoice config show
```

### Installation Testing
```bash
# Test installation script
bash scripts/install.sh

# Test uninstallation script
bash scripts/uninstall.sh

# Verify setup
bash scripts/verify_setup.sh
```

## Architecture Overview

### Technology Stack
- **faster-whisper** (4x faster than openai-whisper) - Transcription engine with VAD support
- **sounddevice + portaudio** - Concurrent audio recording
- **ollama** - Local AI enhancement (summaries, tags, titles)
- **click** - CLI framework
- **rich** - Terminal UI (progress indicators, formatting)
- **pyyaml** - Configuration management
- **pytest** - Testing framework

### Key Design Patterns

#### Zero Data Loss Architecture
**Critical:** Files are created immediately when recording starts, not at the end. All transcript operations use atomic writes to prevent corruption.

```python
# CORRECT: Create file first
filepath, tid = create_transcript()  # File exists NOW
start_audio_capture()                # Then start recording

# WRONG: Create file at end
start_audio_capture()                # Recording starts
save_transcript()                    # ← Crash here = total data loss
```

#### Configuration Hierarchy
Configuration loads in this order (later overrides earlier):
1. Hardcoded defaults (in `core/config.py`)
2. User config file (`~/.config/rejoice/config.yaml`)
3. Environment variables (`.env` file or shell)
4. Command-line flags

#### Test-Driven Development (TDD)
**Required workflow for all new features:**
1. Write failing test first
2. Implement minimal code to pass
3. Refactor while keeping tests green
4. Coverage target: 90%+

### Directory Structure
```
src/rejoice/
├── __init__.py           # Version, exports
├── __main__.py           # CLI entry point
├── cli/                  # CLI commands
│   ├── commands.py       # Main command definitions
│   └── config_commands.py # Config subcommands
├── core/                 # Core functionality
│   ├── config.py         # Configuration system
│   └── logging.py        # Logging setup
├── audio/                # Audio recording (Phase 2)
├── transcription/        # Whisper integration (Phase 3)
├── transcript/           # MD file management
├── ai/                   # Ollama integration (Phase 5)
├── utils/                # Utilities
└── exceptions.py         # Custom exceptions

tests/
├── unit/                 # Fast, isolated tests
├── integration/          # Multi-component tests
├── e2e/                  # Full system tests
├── fixtures/             # Test data
└── conftest.py           # Pytest configuration
```

## Critical Implementation Rules

### 1. Never Lose Data
- Create transcript files immediately when recording starts
- Use atomic writes for all file operations (write to temp, then rename)
- Append transcription segments incrementally, not all at once
- Files must survive crashes, interruptions, power loss

### 2. Respect the "Slim" Mandate
**Never add these:**
- GUI or web interface (terminal only)
- Cloud services or remote APIs
- Database (flat markdown files only)
- Complex plugin systems

### 3. Testing Requirements
- Write tests BEFORE implementation (TDD)
- Unit test coverage: 90%+ required
- Integration tests for all user flows
- E2E tests for critical paths (recording, transcription, AI)
- All new features must have tests

### 4. Configuration Management
- All settings in `~/.config/rejoice/config.yaml`
- Support environment variable overrides (prefix: `REJOICE_`)
- Validate configuration on load
- Provide clear error messages for invalid config

### 5. Error Handling
- Catch all exceptions gracefully
- Provide helpful, actionable error messages
- Suggest fixes when possible
- No stack traces in normal mode (only in `--debug`)

## Important Implementation Details

### Whisper Model Requirements
- **Sample rate:** Must be 16kHz (Whisper requirement)
- **Channels:** Mono (1 channel)
- **VAD (Voice Activity Detection):** Enabled by default to handle silence
- **Model sizes:** tiny, base, small, medium, large (default: medium)

### File Naming Convention
```
transcript_YYYYMMDD_ID.md
```
- Example: `transcript_20241217_000001.md`
- ID is 6-digit zero-padded sequential number
- Later renamed with AI-generated title slug

### Frontmatter Structure
Every transcript starts with YAML frontmatter:
```yaml
---
id: '000001'
type: voice-note
status: recording  # or 'completed', 'processing'
created: 2024-12-17 14:30
language: auto
tags: []
summary: ""
---
```

### Concurrent Audio Access
Uses sounddevice + portaudio to allow microphone access while other apps (Zoom, browsers, Spotify) are running. This is a core requirement.

## Development Workflow

### Adding a New Feature
1. Check `docs/BACKLOG.md` for the user story
2. Write tests first (`tests/unit/test_<feature>.py`)
3. Run tests (should fail): `pytest tests/unit/test_<feature>.py -v`
4. Implement minimal code in `src/rejoice/`
5. Run tests again (should pass)
6. Update story status in BACKLOG.md
7. Commit with clear message

### Working with Configuration
```python
from rejoice.core.config import load_config

# Load config (defaults + user file + env vars)
config = load_config()

# Access settings
model = config.transcription.model
save_path = config.output.save_path

# Config is validated on load - invalid settings raise ConfigError
```

### Using the Logger
```python
from rejoice.core.logging import get_logger

logger = get_logger(__name__)

logger.debug("Detailed debugging info")
logger.info("Normal operation")
logger.warning("Something concerning")
logger.error("Error occurred")
```

### Creating Custom Exceptions
```python
from rejoice.exceptions import RejoiceError

# Raise with helpful message and suggestion
raise RejoiceError(
    "Recording not found",
    suggestion="Try 'rec list' to see available recordings"
)
```

## Common Development Tasks

### Add a New CLI Command
1. Edit `src/rejoice/cli/commands.py`
2. Add `@click.command()` decorated function
3. Add to main group: `main.add_command(new_command)`
4. Write tests in `tests/unit/test_cli.py`
5. Update help text

### Add a New Configuration Option
1. Edit appropriate `@dataclass` in `src/rejoice/core/config.py`
2. Add validation in `Config.validate()` if needed
3. Update `get_default_config()` with default value
4. Add env var mapping in `load_env_overrides()`
5. Write tests in `tests/unit/test_config.py`
6. Document in `docs/CONFIGURATION.md` (when created)

### Add a New Test Fixture
1. Edit `tests/conftest.py`
2. Use `@pytest.fixture` decorator
3. Make available to all tests automatically

## Project Status & Roadmap

### Completed (✅)
- Phase 0: Installation & Environment Setup
  - Development environment
  - Installation script (scripts/install.sh)
  - Uninstallation script (scripts/uninstall.sh)
- Phase 1: Foundation & Project Setup
  - Project structure
  - Package configuration (pyproject.toml)
  - Testing framework (pytest)
  - Configuration system
  - CLI framework (click)
  - Logging system
  - Audio device detection ([R-001])

### In Progress (🚧)
- Phase 2: Core Recording System
  - Audio capture implementation
  - Transcript file management
  - Recording controls (start, stop, cancel)

### Upcoming Phases
- Phase 3: Transcription System (faster-whisper integration)
- Phase 4: Advanced Features (speaker diarization, timestamps)
- Phase 5: AI Enhancement (Ollama integration)
- Phase 6: User Commands (list, view, search, export)
- Phase 7: Configuration & Settings
- Phase 8: Polish & Release

See `docs/BACKLOG.md` for complete story list and status.

## Key Files to Reference

### Documentation
- `docs/VISION.md` - Product philosophy, scope, design principles
- `docs/REWRITE_PLAN.md` - Technical architecture, detailed specs
- `docs/BACKLOG.md` - User stories, task tracking (3400+ lines)
- `docs/DEVELOPMENT.md` - Development workflow guide
- `README.md` - Quick start, overview

### Core Implementation
- `src/rejoice/core/config.py` - Configuration system (hierarchical loading)
- `src/rejoice/cli/commands.py` - CLI command definitions
- `src/rejoice/exceptions.py` - Custom exception classes
- `pyproject.toml` - Package config, dependencies, tool settings

### Installation
- `scripts/install.sh` - One-command installation (macOS/Linux)
- `scripts/uninstall.sh` - Clean removal script
- `scripts/verify_setup.sh` - Installation verification

### Testing
- `tests/conftest.py` - Pytest configuration and fixtures
- `tests/unit/test_config.py` - Config system tests (good example)

## Special Notes

### Virtual Environment Isolation
Rejoice installs to `~/.rejoice/venv` for complete isolation from system Python. The `rec` command is a shell alias that directly calls the venv binary.

### AI-Generated Content
- After transcription completes, Ollama (if available) generates:
  - Descriptive title (3-7 words)
  - Summary (2-3 sentences)
  - Tags (3-7 relevant tags)
  - Action items and questions (optional)
- File is renamed with AI-generated title slug
- All prompts are customizable in `~/.config/rejoice/prompts/`

### Whisper Model Selection
- **tiny** (75MB): Very fast, good accuracy
- **base** (142MB): Fast, better accuracy
- **small** (466MB): Medium speed, great accuracy (default)
- **medium** (1.5GB): Slow, excellent accuracy
- **large** (2.9GB): Very slow, best accuracy

### Platform Support
- **macOS:** Full support (both Intel and Apple Silicon)
- **Linux:** Full support (Ubuntu, Debian, Fedora tested)
- **Windows:** Not yet supported (planned for v3.0+)

## Getting Help

### When Making Changes
1. Read `docs/VISION.md` to understand core principles
2. Check `docs/REWRITE_PLAN.md` for detailed specs
3. Look at existing tests for patterns
4. Follow TDD: tests first, then implementation
5. Run pre-commit checks before committing

### Common Issues
- **Import errors:** Make sure package is installed in editable mode (`pip install -e .`)
- **Test failures:** Check you've activated the venv (`source venv/bin/activate`)
- **Config not loading:** Ensure `~/.config/rejoice/config.yaml` is valid YAML
- **Audio device errors:** Run `rec config list-mics` to see available devices

### Decision Framework
When adding features, ask:
1. Does this align with "Slim"? (No GUI, no cloud, minimal complexity)
2. Does this improve data integrity? (High priority)
3. Does this make the tool simpler? (Fewer steps = better)
4. Is this already solved by existing tools? (Don't reinvent)
5. Would this violate local-first principle? (Must work offline)

## Remember
- **Data integrity is non-negotiable** - Never lose a transcript
- **Simplicity over features** - Delete more than you add
- **Local-first, local-only** - No internet required
- **Transparency over magic** - User always knows what's happening
- **Boring reliability** - Code that just works is better than clever code

For more details, see the comprehensive documentation in `docs/`.

---
> Source: [Lenniott/rejoice-slim-v2](https://github.com/Lenniott/rejoice-slim-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
