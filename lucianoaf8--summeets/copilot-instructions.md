## summeets

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Summeets** is a production-grade Python monorepo for transcribing and summarizing meetings with speaker diarization. It provides a shared processing core exposed through CLI and TUI interfaces:

### Key Features
- **Audio Processing**: FFmpeg-based normalization, extraction, and format conversion
- **Transcription**: Replicate's `thomasmol/whisper-diarization` (Whisper v3 + Pyannote) for speaker-aware transcription
- **Summarization**: Map-reduce + Chain-of-Density summarization with OpenAI GPT-4o or Anthropic Claude
- **Dual Interface**: Full-featured CLI (`summeets`) and Textual TUI (`summeets tui`) sharing the same core
- **Production Ready**: Structured logging, configuration management, error handling, comprehensive test suite

### Architecture
The project follows clean architecture principles with clear separation of concerns:
- **Source Module** (`src/`): Shared business logic, models, and utilities
- **CLI Interface** (`cli/`): Typer-based command-line interface + Textual TUI
- **Archive** (`archive/`): Deprecated components (including legacy Electron GUI)
- **Main Entry Point**: Router that launches CLI

## Dependencies

Install the package and its dependencies:
```bash
pip install -e .
```

Core dependencies include:
- `typer` - CLI framework
- `textual` - TUI framework for terminal-based graphical interface
- `pydantic` + `pydantic-settings` - Configuration and validation
- `openai` + `anthropic` - LLM providers for summarization
- `replicate` - Transcription API
- `tenacity` - Retry logic
- `alive-progress` - Progress indicators
- `rich` - Terminal formatting

External dependencies:
- `ffmpeg` and `ffprobe` (optional but recommended) - For audio/video processing

## Environment Setup

1. Create a `.env` file with your API keys:
   ```env
   # LLM Provider (openai or anthropic)
   LLM_PROVIDER=openai
   LLM_MODEL=gpt-4o-mini
   
   # API Keys (at least one required)
   OPENAI_API_KEY=sk-...
   ANTHROPIC_API_KEY=sk-ant-...
   REPLICATE_API_TOKEN=r8_...
   
   # Optional settings
   SUMMARY_MAX_OUTPUT_TOKENS=3000
   SUMMARY_CHUNK_SECONDS=1800
   SUMMARY_COD_PASSES=2
   ```

2. Install the package:
   ```bash
   pip install -e .
   ```

## Running the Tool

### CLI Interface
```bash
# Complete pipeline (transcribe + summarize)
summeets process /path/to/meeting.m4a

# Individual steps
summeets transcribe /path/to/audio.m4a
summeets summarize out/audio.json --provider openai --model gpt-4o

# Audio processing
summeets normalize input.mkv output.mkv
summeets extract input.mkv output.m4a --codec aac
summeets probe input.mkv

# View configuration
summeets config
```

### TUI Interface
```bash
# Launch the Textual-based TUI
summeets tui
```

The TUI provides a graphical terminal interface for running workflows with real-time progress tracking.

### Direct Execution
```bash
# Default: launches CLI
python main.py

# Or via installed package
summeets [command]
```

## Code Architecture

### Clean Architecture Structure
```
summeets/
├─ src/                     # Shared processing core
│  ├─ models.py             # Pydantic data models & job tracking
│  ├─ workflow.py           # Flexible workflow engine
│  ├─ audio/                # Audio/video processing
│  │  ├─ ffmpeg_ops.py      # FFmpeg operations (audio + video)
│  │  ├─ selection.py       # Audio file selection logic
│  │  └─ compression.py     # Audio compression utilities
│  ├─ providers/            # LLM clients
│  │  ├─ openai_client.py   # OpenAI API client
│  │  └─ anthropic_client.py # Anthropic API client
│  ├─ transcribe/           # Transcription pipeline
│  │  ├─ pipeline.py        # Main transcription logic
│  │  ├─ formatting.py      # Output formatting (JSON, SRT)
│  │  └─ replicate_api.py   # Replicate API integration
│  ├─ summarize/            # Summarization pipeline
│  │  └─ pipeline.py        # Map-reduce + COD summarization
│  ├─ services/             # Dependency injection layer
│  │  ├─ interfaces.py      # Abstract service contracts
│  │  ├─ container.py       # Service container & resolution
│  │  └─ implementations.py # Concrete service wrappers
│  └─ utils/                # Utility modules
│     ├─ config.py          # Pydantic settings management
│     ├─ logging.py         # Structured logging setup
│     ├─ fsio.py            # File system operations
│     ├─ jobs.py            # Job management
│     ├─ cache.py           # Caching utilities
│     ├─ security.py        # Security utilities
│     ├─ validation.py      # Input validation (video/audio/transcript)
│     └─ exceptions.py      # Custom exceptions
├─ cli/                     # CLI interface
│  ├─ app.py                # Typer CLI commands
│  └─ tui/                  # Textual TUI application
│     ├─ app.py             # Main TUI app
│     ├─ widgets.py         # Custom widgets
│     └─ demo.py            # Demo/test mode
├─ archive/                 # Deprecated components
│  └─ electron_gui/         # Legacy Electron GUI (deprecated)
├─ main.py                  # Entry point router
├─ data/                    # Organized data storage
│  ├─ input/                # Input files (by date)
│  ├─ output/               # Processing results (by date/type)
│  ├─ temp/                 # Temporary files
│  └─ jobs/                 # Job state persistence
├─ tests/                   # Comprehensive test suite
│  ├─ unit/                 # Unit tests
│  ├─ integration/          # Integration tests
│  └─ conftest.py           # Test configuration
└─ scripts/win_dev.ps1      # Development setup script
```

### Core Components

- **Workflow Engine** (`src.workflow`): Flexible pipeline supporting video/audio/transcript inputs
- **Audio/Video Processing** (`src.audio.ffmpeg_ops`): FFmpeg-based operations including:
  - Video to audio extraction with quality settings
  - Audio normalization and volume adjustment
  - Format conversion (m4a, mp3, wav, flac, ogg)
  - Video metadata extraction
- **Audio Selection** (`src.audio.selection`): Intelligently selects highest quality audio from directories
- **Transcription Pipeline** (`src.transcribe.pipeline`): Replicate API integration with progress tracking
- **Summarization Pipeline** (`src.summarize.pipeline`): Map-reduce chunking + Chain-of-Density refinement
- **LLM Providers** (`src.providers`): Unified interface for OpenAI and Anthropic APIs
- **Configuration** (`src.utils.config`): Pydantic-based settings with environment variable support
- **Job Management** (`src.utils.jobs`): State persistence and progress tracking
- **Validation** (`src.utils.validation`): Input validation for video, audio, and transcript files
- **Service Layer** (`src.services`): Dependency injection container (see below)

### Service Layer / Dependency Injection

The project uses a lightweight DI container for core services:

- **Interfaces** (`src/services/interfaces.py`): `AudioProcessorInterface`, `TranscriberInterface`, `SummarizerInterface` — abstract contracts
- **Implementations** (`src/services/implementations.py`): `FFmpegAudioProcessor`, `ReplicateTranscriberService`, `LLMSummarizer` — concrete wrappers around existing modules
- **Container** (`src/services/container.py`): `ServiceContainer` with singleton/factory/instance registration, thread-safe resolution

Usage pattern:
```python
from src.services import get_container, register_default_services
register_default_services()  # idempotent
container = get_container()
audio = container.get_audio_processor()
```

Adoption status: `WorkflowEngine` uses the container for service initialization. CLI commands still call module-level functions directly for simple operations (probe, normalize).

### Data Models

- `TranscriptWord`: Individual word with timing and confidence
- `TranscriptSegment`: Text segment with speaker attribution and word-level timing
- `TranscriptData`: Complete transcript with metadata
- `SummaryData`: Structured summary with sections and metadata
- `WorkflowConfig`: Configuration for flexible workflow execution
- `WorkflowStep`: Individual workflow step with conditional execution
- `JobData`: Job state tracking for long-running operations

## Supported File Formats

### Audio Formats
Ranked by preference: `.m4a`, `.flac`, `.wav`, `.mka`, `.ogg`, `.mp3`, `.webm`

### Video Formats
Supported: `.mp4`, `.mkv`, `.avi`, `.mov`, `.wmv`, `.flv`, `.webm`, `.m4v`

### Transcript Formats
Supported: `.json`, `.txt`

The tool automatically detects file type and adjusts the workflow accordingly:
- **Video files**: Extract audio → Process → Transcribe → Summarize
- **Audio files**: Process → Transcribe → Summarize
- **Transcript files**: Summarize only

## Development

### Code Quality Tools
```bash
# Type checking
mypy src/ cli/

# Linting
ruff check .

# Testing
python -m pytest tests/
```

### Entry Points
- `summeets` command maps to `main:main` function
- CLI and TUI share the same core processing logic
- Configuration is centralized through Pydantic settings
- TUI runs via `summeets tui` command

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lucianoaf8) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
