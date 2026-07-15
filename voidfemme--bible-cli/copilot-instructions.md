## bible-cli

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a fully-featured Python-based Bible CLI application that allows users to retrieve Bible passages in multiple translations including Greek and Hebrew texts. The application is designed for Unix pipeline integration and provides flexible output formats.

Key features:
- Support for 20+ Bible translations (KJV, ESV, NIV, NASB, etc.)
- Original language support (Greek SBLGNT, Hebrew WLC)
- Multiple output formats (plain, JSON, CSV, TSV, verse-per-line)
- Pipeline-friendly design for bash integration
- Comprehensive reference parsing with abbreviation support
- AI-powered semantic search using sentence transformers

## Project Structure

- `src/bible_cli.py` - Main CLI application with Click framework
- `src/semantic_search.py` - AI-powered semantic search functionality
- `main.py` - Entry point for the application
- `tests/test_bible_cli.py` - Unit tests for core functionality
- `tests/test_semantic_search.py` - Tests for semantic search
- `setup.py` - Package installation configuration
- `requirements.txt` - Python dependencies (requests, click, rich, sentence-transformers)

## Development Commands

```bash
# Quick install (makes 'bible' command available globally)
./install.sh

# Development setup
make dev              # Set up development environment
make install          # Install globally
make uninstall        # Remove global installation
make test             # Run tests
make clean            # Clean build artifacts

# Manual setup
python -m venv venv
source venv/bin/activate  # Linux/Mac
pip install -r requirements.txt

# Run the application
bible "John 3:16"                    # If installed globally
python main.py "John 3:16"           # If running locally
bible --help
bible --list-translations

# Run tests
python -m unittest tests.test_bible_cli -v

# Test CLI functionality
bible "John 3:16" -t esv --format json
bible "psalm 23" --format plain | wc -w
bible "genesis 1:1" --hebrew
```

## API Integration

The application uses multiple Bible APIs:
- **bible-api.com** - Primary API for common English translations
- **getBible API** - Fallback API with broader translation support
- Supports rate limiting and error handling

## Architecture Notes

- Uses Click framework for command-line interface
- Rich library for colored terminal output
- Requests library for HTTP API calls
- Modular design with separate functions for:
  - Reference normalization and parsing
  - API fetching with fallback support
  - Output formatting for different formats
  - Error handling and validation

## CLI Usage Examples

```bash
# Basic usage
bible "John 3:16"                    # KJV (default)
bible "John 3:16" -t esv             # English Standard Version
bible "jn 3:16" -t niv               # Abbreviations supported

# Original languages
bible "John 1:1" --greek             # Greek text
bible "Genesis 1:1" --hebrew         # Hebrew text

# Output formats
bible "John 3:16" --format json      # JSON output
bible "John 3:16" -q                 # Plain text (quiet)
bible "Matthew 5:3-12" --format csv  # CSV format

# Semantic search (AI-powered)
bible --init-search                  # Initialize search corpus (one-time)
bible "love your enemies" --search   # Find verses by meaning
bible "faith and works" --search --format json

# Pipeline integration
bible "Psalm 23" -q | wc -w          # Count words
bible "John 3:16" --format json | jq '.verses[].text'
bible "peace" --search --format json | jq -r '.[] | .reference + ": " + .text'
```

## Testing

The test suite covers:
- Reference normalization
- API fetching (with mocking)
- Output formatting for all formats
- Error handling scenarios

Run tests with: `python -m unittest tests.test_bible_cli -v`

---
> Source: [voidfemme/bible-cli](https://github.com/voidfemme/bible-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
