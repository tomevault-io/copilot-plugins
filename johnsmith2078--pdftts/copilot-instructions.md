## pdftts

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a PyQt6-based PDF reader application with integrated OCR (Optical Character Recognition) and TTS (Text-to-Speech) functionality. The application allows users to view PDF files, perform OCR on pages, and have the recognized text read aloud with visual highlighting.

## Key Components

- **main.py**: Main application window with PDF viewing controls, OCR/TTS orchestration, and session management
- **pdf_viewer.py**: PDF display widget with zoom, navigation, and text highlighting capabilities
- **config.json**: Session persistence (last opened file and page positions)

## Development Commands

### Installation & Setup
```bash
pip install -r requirements.txt
```

### Running the Application
```bash
python main.py
```

### Dependencies
- PyQt6 (GUI framework)
- PyMuPDF (PDF processing)
- PaddleOCR (Chinese OCR, auto-downloads models on first run)
- pyttsx3 (Text-to-speech)
- OpenCV, Pillow (image processing)

## Architecture Overview

### Threading Model
- **Main Thread**: GUI operations, TTS engine management
- **TTSWorker (QThread)**: Manages sequential text-to-speech playback
- **OCR Thread**: Background OCR processing to avoid UI blocking

### State Management
- **TTSState**: Centralized TTS state tracking (current segment, stop requests)
- **Session Management**: Automatic save/restore of opened files and page positions

### Core Features
- **PDF Viewing**: Multi-page navigation with zoom and scroll
- **OCR Integration**: 200 DPI processing with confidence filtering (>0.5)
- **TTS with Highlighting**: Sequential text reading with visual feedback
- **Coordinate Transformation**: Converts OCR coordinates to display coordinates based on zoom level

### Signal-Slot Architecture
Key signals:
- `page_changed`: Emitted when PDF page changes
- `text_segment_started/finished`: TTS progress tracking
- `request_speak`: Cross-thread TTS requests

## Important Implementation Details

### TTS Engine Management
- TTS engine is initialized in main thread only
- Complete re-initialization after stop operations to prevent driver issues
- Uses empty string ping tests for engine validation

### OCR Text Processing
- Intelligent text block sorting (top-to-bottom, left-to-right)
- Confidence-based filtering
- Empty/whitespace segment removal before TTS

### Coordinate System
- OCR uses 200 DPI, display uses current zoom factor
- Coordinate transformation: `display_scale / ocr_scale`
- Highlight rendering uses QPainter with semi-transparent overlays

### Error Handling
- Graceful degradation when OCR/TTS engines fail
- Worker thread cleanup on application close
- Invalid coordinate validation for highlights

## Session Management

The application automatically saves:
- Last opened file path
- Page positions for each opened file
- Session data persisted to `config.json`

## Keyboard Shortcuts

- `Ctrl+O`: Open file
- `Ctrl+R`, `F9`: OCR and read current page
- `Ctrl+T`: Stop reading
- `Ctrl+±`: Zoom in/out
- Arrow keys, Page Up/Down: Navigate pages

---
> Source: [johnsmith2078/pdftts](https://github.com/johnsmith2078/pdftts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
