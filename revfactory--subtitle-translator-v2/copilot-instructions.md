## subtitle-translator-v2

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a YouTube subtitle translation tool that extracts subtitles from videos and translates them from English to Korean using Claude API. The project includes both command-line scripts and a PyQt6 GUI application.

## Core Architecture

### Main Components

1. **subtitle.py** - Core translation engine with modular class structure:
   - `SubtitleTranslationConfig` - Configuration management with file and CLI argument support
   - `SubtitleFileHandler` - SRT file I/O operations 
   - `SubtitleProcessor` - Subtitle parsing, batching, and timestamp validation
   - `ClaudeTranslator` - Claude API integration for translation
   - `SubtitleTranslator` - Main orchestration class with parallel processing

2. **youtube_subtitle.py** - Simple CLI workflow script that chains:
   - YouTube video download (yt-dlp)
   - Subtitle extraction (AssemblyAI)
   - Translation (calls subtitle.py)

3. **app/app.py** - PyQt6 GUI application with tabbed interface:
   - YouTube download tab with progress tracking
   - Subtitle extraction tab
   - Translation tab with progress visualization
   - Settings tab for configuration

### Key Dependencies

- **anthropic**: Claude API client for translation
- **yt-dlp**: YouTube video downloading
- **assemblyai**: CLI tool for subtitle extraction
- **PyQt6**: GUI framework (app only)
- **tqdm**: Progress bars for CLI

## Common Commands

### Development & Testing
```bash
# Install dependencies
pip install anthropic tqdm PyQt6 py2app

# Core translation
python subtitle.py path/to/file.srt

# YouTube workflow
python youtube_subtitle.py "https://youtube.com/watch?v=VIDEO_ID"

# GUI application
python app/app.py

# Generate default config
python subtitle.py --gen-config
```

### GUI App Packaging (macOS)
```bash
cd app
python setup.py py2app
```

## Configuration System

- **config.json** - Main configuration file with translation settings
- Uses environment variables for API keys: `ANTHROPIC_API_KEY`, `ASSEMBLYAI_API_KEY`
- Command-line arguments override config file settings
- GUI settings are persisted to config.json

### Key Settings
- `model`: Claude model identifier (default: claude-sonnet-4-20250514)
- `batch_size`: Subtitles per translation request (default: 5)
- `max_workers`: Parallel translation threads (default: 3)
- Token cost settings for billing calculation

## Translation Pipeline

1. **File Processing**: SRT parsing into individual subtitle blocks
2. **Batching**: Grouping subtitles for efficient API usage
3. **Parallel Translation**: ThreadPoolExecutor for concurrent API calls
4. **Post-processing**: 
   - Subtitle renumbering
   - Timestamp overlap detection and adjustment
5. **Output**: Korean SRT with "_ko" suffix

## Important Implementation Details

- **Timestamp Validation**: Automatically fixes overlapping subtitle timestamps
- **Error Handling**: Retry logic with exponential backoff for API failures
- **Progress Tracking**: Real-time progress bars in both CLI and GUI
- **File Naming**: Consistent patterns (removes YouTube video IDs, adds language suffixes)
- **Cost Tracking**: Token usage monitoring and cost calculation

## GUI Threading Architecture

- **TranslationThread**: Handles subtitle translation with progress callbacks
- **YoutubeDownloadThread**: Downloads video/audio with real-time progress parsing
- **ExtractSubtitleThread**: Runs AssemblyAI subtitle extraction
- All threads use PyQt signals for safe UI updates

## External Tool Requirements

- **yt-dlp**: Must be installed globally for YouTube downloads
- **assemblyai CLI**: Required for subtitle extraction
- Both tools are called via subprocess with output parsing

---
> Source: [revfactory/subtitle-translator-v2](https://github.com/revfactory/subtitle-translator-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
