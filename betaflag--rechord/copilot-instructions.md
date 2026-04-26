## rechord

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Rechord is a Python CLI tool that enhances guitar tablature files using Google's Gemini AI. It converts raw tab text into well-formatted markdown with YAML front matter containing song metadata.

## Key Commands

### Development Setup
```bash
# Create and activate virtual environment
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies and package in editable mode
pip install -r requirements.txt
pip install -e .

# For development with testing and linting tools
pip install -e ".[dev]"
```

### Running Tests
```bash
# Run all tests with verbose output
pytest tests/ -v

# Run a specific test file
pytest tests/test_formatter.py -v

# Run a specific test
pytest tests/test_formatter.py::TestTabFormatter::test_parse_tab_with_front_matter -v

# Run with coverage
pytest tests/ --cov=rechord
```

### Code Quality
```bash
# Check for linting issues
ruff check .

# Auto-fix linting issues
ruff check . --fix

# Format code
ruff format .
```

### Using the CLI
```bash
# Basic enhancement (outputs as artist-song-name.md)
rechord enhance input_tab.txt

# With validation
rechord enhance input_tab.txt --validate

# Batch processing with glob patterns
rechord batch "*.txt" --output-dir ./enhanced
rechord batch "**/*.txt" --output-dir ./enhanced --recursive
rechord batch "{*.txt,*.tab}" --output-dir ./enhanced --validate

# Validate format
rechord validate enhanced_tab.md
```

## Architecture

### Core Components

**GeminiClient** (`rechord/gemini_client.py`)
- Uses the new `google-genai` SDK (imported as `from google import genai`)
- Handles all Gemini API interactions with retry logic
- Uses `genai.types.GenerateContentConfig` for configuration
- Contains the enhancement prompt that defines output format
- Single enhancement mode - no basic/advanced distinction
- Improved prompt (v3) with:
  - Clear output format specification upfront
  - Roman numeral conversion for capo (VIII→8)
  - Explicit rules for tab block usage (6-line notation only)
  - Chord placement on ALL lyrics lines
  - Enhanced French language support:
    * Preserves accents (é, è, ê, à, ç, ô, û)
    * Maintains contractions (j'ai, l'amour, d'être)
    * Handles Quebec French colloquialisms
    * Recognizes French musical terms (En boucle, Couplet, Refrain)
  - Proper chord loop/progression formatting
  - No title header in body content

**TabFormatter** (`rechord/formatter.py`)
- Parses tabs with YAML front matter
- Validates required metadata (title, artist)
- Formats parsed data back to markdown
- Simple structure: metadata + body (no sections)
- Handles both YAML front matter and plain text input

**CLI** (`rechord/cli.py`)
- Three main commands: enhance, validate, batch
- Uses Click framework for command parsing
- Loads API key from environment or `.env` file
- Auto-generates output filename as `artist-song-name.md` from metadata
- Batch command:
  - Accepts glob patterns directly (e.g., "*.txt", "**/*.txt")
  - Supports recursive patterns with --recursive flag
  - Supports brace expansion (e.g., "{*.txt,*.tab}")
  - Creates output directory if needed
  - Handles duplicate filenames with counter
  - Optional validation for each file

### Output Format

The enhanced tabs follow this simplified structure:
1. YAML front matter with metadata (NO title header after front matter)
2. Tab notation (if present) in ```tab blocks for 6-line tablature ONLY
3. Lyrics with chords positioned above each line

Key format rules:
- No chord definitions section
- No verse/chorus/section markers
- No title header in body (already in YAML)
- Capo info in YAML only (e.g., `capo: 8`)
- Chord progressions as plain text, not in tab blocks

### API Integration

The project uses Google's new `google-genai` library (v1.31.0+) with the updated API:
- Client initialization: `genai.Client(api_key=api_key)`
- Content generation: `client.models.generate_content(model=..., contents=..., config=...)`
- Config type: `genai.types.GenerateContentConfig`

## Important Notes

- Always ensure GEMINI_API_KEY is set before running enhancement commands
- The formatter expects tabs to either have YAML front matter or be plain text
- Tests use pytest and all must pass before committing changes
- Code is formatted with ruff (88 char line length, double quotes)
- Required metadata fields: title and artist (validation will fail without these)
- Default model: `gemini-1.5-pro` with temperature 0.3 for consistent formatting

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/betaflag) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
