## ai-excel-translator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI-powered Excel translation tool that automatically translates content between Vietnamese and Japanese using Google's Gemini API. The tool preserves Excel formatting while translating text in cells and shapes (TextBox, WordArt, etc.).

## Core Architecture

- **Main Application**: `trans-excel2.py` - Single-file Python application that handles the entire translation workflow
- **API Integration**: Uses OpenAI-compatible client to connect to Google Gemini API (configurable for other providers)
- **Excel Processing**: Leverages xlwings library for direct Excel COM automation to preserve formatting
- **Batch Processing**: Processes text in configurable batches to optimize API usage and respect rate limits
- **Multi-format Support**: Handles both cell content and shape text with multiple fallback methods

## Key Development Commands

```bash
# Install dependencies
pip install -r trans-excel-requirements.txt

# Run translation (Vietnamese to Japanese - default)
python trans-excel2.py

# Run translation (Japanese to Vietnamese)
python trans-excel2.py --to vi

# Process multiple files in input directory
python trans-excel2.py --to ja
```

## Configuration

### Environment Variables
- **Required**: `GEMINI_API_KEY` in `.env` file for Google Gemini API access
- **Template**: Use `.env.sample` as template for environment setup

### Key Configuration Constants
- `API_DELAY = 2`: Seconds between API calls to respect rate limits
- `BATCH_SIZE = 100`: Maximum text segments per API call
- API Provider: Configured for Gemini 2.0 Flash Lite with OpenAI-compatible interface

### System Prompt Customization
- **File**: `trans-excel-system-prompt.txt` (auto-generated on first run)
- **Purpose**: Controls translation quality and behavior
- **Customization**: Modify for different industries (medical, legal, technical)

## Input/Output Structure

```
/input/           # Place Excel files here (.xlsx, .xls)
/output/          # Translated files appear here
trans-excel2.py    # Main application
trans-excel-system-prompt.txt  # System prompt (auto-generated)
```

## Development Guidelines

### Adding New Language Support
1. Update language pair definitions in `translate_batch()` function
2. Modify argument parser in `main()` to accept new language codes
3. Update language display logic in main function
4. Consider updating system prompt for new language expertise

### API Provider Configuration
The application uses OpenAI-compatible interface. To change providers:
- Update `base_url` in OpenAI client initialization
- Change `model` parameter in `translate_batch()`
- Update environment variable names as needed

### Excel Processing Architecture
The tool uses multiple fallback methods for extracting/updating text in shapes:
1. `TextFrame.Characters().Text` - Primary method
2. `TextFrame2.TextRange.Text` - Alternative method
3. `AlternativeText` - For shapes with alternative text
4. `OLEFormat.Object.Text` - For OLE embedded objects
5. `TextEffect.Text` - For WordArt objects

## Error Handling & Robustness

- **Graceful degradation**: Falls back to original text if translation fails
- **Batch size validation**: Ensures translated result count matches input count
- **Excel automation safety**: Proper cleanup of Excel COM objects and processes
- **Shape processing resilience**: Multiple fallback methods for different shape types
- **Rate limit management**: Built-in delays between API calls

## Dependencies

- `openai>=1.0.0`: API client library
- `xlwings>=0.30.0`: Excel COM automation
- `python-dotenv>=1.0.0`: Environment variable management
- `pathlib>=1.0.1`: Path handling utilities

## Platform Requirements

- **Windows/macOS only**: Requires Microsoft Excel installation
- **Python 3.7+**: Minimum Python version
- **Excel COM integration**: xlwings requires Excel to be installed locally

---
> Source: [hoangduong92/ai-excel-translator](https://github.com/hoangduong92/ai-excel-translator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
