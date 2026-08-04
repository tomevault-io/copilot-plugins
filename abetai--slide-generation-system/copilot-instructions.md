## slide-generation-system

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Japanese lecture slide generation system using Streamlit and the Claude API. The system provides three main workflows:

1. **Text → Human-readable outline**: Uses Claude to generate editable text outlines
2. **Outline → PowerPoint**: Converts finalized outlines to JSON format and generates PPTX slides  
3. **PowerPoint → Speaker Notes**: Analyzes completed slides and generates presenter notes using Claude Vision API

## Architecture

The system follows a modular pipeline architecture:

```
Text Input
    ↓
generate_outline.py (text → outline with Claude)
    ↓
[Human review/editing phase]
    ↓
outline_to_json.py (outline → JSON with Claude)
    ↓
generate_slides.py (JSON → PowerPoint using python-pptx)

# Alternative workflow for existing presentations:
Completed PPTX
    ↓
generate_speaker_notes.py (PPTX → PDF → images → speaker notes with Claude Vision)
    ↓
Updated PPTX with speaker notes
```

**Key Design Decision**: The human-readable outline format allows for easy manual editing between steps, which is critical for content accuracy in educational materials.

## Core Components

- **app.py**: Streamlit web interface with 3-tab workflow
- **generate_outline.py**: Claude integration for text-to-outline conversion with detail levels ("standard" vs "detailed")
- **outline_to_json.py**: Claude integration for outline-to-JSON conversion with format error correction
- **generate_slides.py**: PowerPoint generation using python-pptx with specific template requirements
- **generate_speaker_notes.py**: Speaker notes generation using Claude Vision API (PPTX → PDF → images → notes → updated PPTX)

## Development Commands

### Setup and Dependencies
```bash
# Install dependencies using uv (preferred)
uv install

# Alternative with pip  
pip install anthropic>=0.73.0 dotenv>=0.9.9 python-pptx>=1.0.2 streamlit>=1.51.0 PyMuPDF>=1.23.0
```

### Running the Application
```bash
# Start Streamlit development server (with uv)
uv run streamlit run app.py

# Or without uv
streamlit run app.py
```

### Environment Variables
```bash
# Required for Claude API access
export ANTHROPIC_API_KEY="your_api_key_here"
```

### Testing Individual Components
```bash
# Test outline generation
uv run python generate_outline.py

# Test JSON conversion 
uv run python outline_to_json.py

# Test slide generation (requires template and JSON)
uv run python generate_slides.py

# Test speaker notes generation (requires existing PPTX file)
uv run python generate_speaker_notes.py input.pptx output.pptx
```

### System Requirements for Speaker Notes Generation

**LibreOffice**: Required for PPTX to PDF conversion
```bash
# macOS (Homebrew)
brew install --cask libreoffice

# macOS (MacPorts) 
sudo port install libreoffice

# Ubuntu/Debian
sudo apt install libreoffice

# The system will automatically detect LibreOffice installation
```

## PowerPoint Template Requirements

The system expects a specific template structure in `スライドテンプレ.pptx`:

- **Layout 0 (main_template)**: Main content slides
  - Placeholder idx=13: Left side (heading)
  - Placeholder idx=14: Right side (content)
- **Layout 1 (start)**: Title slide
  - Placeholder idx=13: Title
- **Layout 2 (end)**: Closing slide

## Claude API Integration

All Claude integrations use the Claude Sonnet model (`claude-sonnet-4-5-20250929`) with specific prompting strategies:

- **Outline generation**: Supports "standard" and "detailed" modes for different content depths
- **JSON conversion**: Includes error correction for human-edited outline formats
- **Speaker notes generation**: Uses Vision API to analyze slide images and generate presenter notes
- **Token limits**: Set to 20000 for outline/JSON, 4000 for speaker notes generation

## File Format Specifications

**Outline Format**:
```
タイトル: [Title]

アジェンダ:
1. [Agenda Item 1]
2. [Agenda Item 2]

---

## 1. [Agenda Item 1]

### スライド1
- Bullet point 1
- Bullet point 2
  - Sub-point

### スライド2
- Content for second slide
```

**JSON Schema**:
```json
{
  "title": "Lecture Title",
  "agenda": ["Item 1", "Item 2"],
  "main": {
    "Item 1": ["Slide 1 content", "Slide 2 content"],
    "Item 2": ["Slide content"]
  }
}
```

## Streamlit Session State Management

Key session variables:
- `outline`: Generated outline text
- `detail_level`: "standard" or "detailed" 
- `final_outline`: Human-confirmed outline
- `json_data`: Converted JSON structure
- `pptx_data`: Generated PowerPoint binary data

## Error Handling Patterns

- Claude API failures are caught and displayed to users
- JSON parsing errors include response debugging
- File I/O operations use proper encoding (utf-8) for Japanese content
- Template validation occurs before slide generation
- LibreOffice subprocess errors are handled for PDF conversion
- PyMuPDF errors are caught during PDF to image conversion
- Temporary file cleanup is performed after processing

---
> Source: [AbeTai/slide-generation-system](https://github.com/AbeTai/slide-generation-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
