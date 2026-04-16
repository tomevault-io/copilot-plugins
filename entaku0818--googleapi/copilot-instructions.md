## googleapi

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Google Cloud Text-to-Speech (TTS) application that converts SSML (Speech Synthesis Markup Language) text into audio files using Google's Text-to-Speech API. The codebase is written in Python and generates MP3 audio files from SSML input.

## Common Development Commands

### Setup and Dependencies
```bash
# Install dependencies
pip install -r requirements.txt

# Create and configure environment
cp .env.example .env
# Edit .env to set GOOGLE_APPLICATION_CREDENTIALS path
```

### Running the Application
```bash
# Run with default SSML samples from ssml_list.json
python ssml_to_speech.py

# The script will automatically use ssml_list.json if it exists,
# otherwise falls back to hardcoded samples
```

## Architecture and Key Components

### Core Module: `ssml_to_speech.py`
- **GoogleTTSGenerator class**: Main class that handles TTS operations
  - Manages Google Cloud TTS client initialization
  - Default voice: ja-JP-Neural2-B (Japanese, Female)
  - Default output format: MP3
  - Methods:
    - `generate_speech()`: Converts single SSML to audio
    - `process_ssml_list()`: Batch processes multiple SSML inputs

### Configuration Files
- **ssml_list.json**: Contains SSML examples with optional voice configurations
  - Each entry can specify: name, ssml, voice parameters
  - Voice parameters include: language_code, name, gender
- **.env**: Stores Google Cloud credentials path (required)
  - GOOGLE_APPLICATION_CREDENTIALS: Path to service account JSON
  - GOOGLE_CLOUD_PROJECT: Optional project ID

### Output Structure
- Audio files are saved to `output/` directory (created automatically)
- File naming: `{name}.mp3` where name comes from ssml_list.json

## Google Cloud Configuration Requirements

1. Active Google Cloud project with Text-to-Speech API enabled
2. Service account with appropriate permissions
3. Downloaded JSON credentials file
4. Credentials path set in .env file

## Available Voice Options

### Japanese Voices
- ja-JP-Neural2-A (Female)
- ja-JP-Neural2-B (Male) - Default
- ja-JP-Neural2-C (Male)
- ja-JP-Neural2-D (Male)

### English Voices
- en-US-Neural2-A (Male)
- en-US-Neural2-C (Female)
- en-US-Neural2-D (Male)
- en-US-Neural2-F (Female)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/entaku0818) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
