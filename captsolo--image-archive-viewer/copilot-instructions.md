## image-archive-viewer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ComicView is a Python-based image archive viewer for viewing images from ZIP, RAR, CBR and CBZ files. The application is built using PyQt5 and provides a fullscreen slideshow interface for browsing image archives, primarily designed for comic book reading and photo collection viewing.

## Project Structure

- `src/image_archive_viewer/` - Main package directory
  - `viewer.py` - Main viewer application and UI components
  - `archive_reader.py` - Archive reading and image processing functionality
  - `logging_setup.py` - Logging configuration
  - `__init__.py` - Package initialization file
- `tests/` - Test suite directory
  - `fixtures/` - Test fixture files and creation scripts
  - `test_*.py` - Unit test modules
  - `conftest.py` - Shared test fixtures and configuration
- `pyproject.toml` - Project configuration, dependencies, and build settings
- `uv.lock` - Dependency lock file (managed by `uv`)

## Compatibility

ComicView works with:
- Linux
- macOS  
- Microsoft Windows

The application will fail reading CBR and RAR files if `unrar` is not installed:
- **macOS and Linux**: `unrar` can be installed using a package manager (e.g. Homebrew on macOS)
- **Windows**: download `unrar` from the [RARLab site](https://www.rarlab.com/rar_add.htm) and put it in a folder on the system search PATH

## Development Commands

### Installation and Setup
```bash
# Install in development mode using uv (preferred)
uv pip install -e .
```

#### Install from PyPI

```bash
# Install as a uv tool for global access
uv tool install image-archive-viewer

# Install from PyPI
pip install image-archive-viewer
```

### Running the Application
```bash
# Run the installed command (opens file dialog)
comic_view

# Run with verbose logging (logs to console and image_archive_viewer.log)
comic_view -v
```

### Testing
The project has a test suite configured for pytest in `pyproject.toml` with `pythonpath = "src"`:
```bash
# Run all automated tests (safe for CI/headless)
pytest

# Run tests with verbose output
pytest -v

# Run specific test module
pytest tests/test_archive_reader.py

# Generate test fixtures (creates ZIP/CBZ archives)
cd tests/fixtures && python create_test_fixtures.py
```

#### Test Structure
- **`test_basic.py`**: Package imports and fixture validation
- **`test_archive_reader.py`**: Archive reading functionality (ZIP/CBZ/RAR/CBR)
- **Fixtures**: Sample archives in `tests/fixtures/` (ZIP, CBZ, RAR formats)

### Building and Distribution
```bash
# Build package for distribution
python -m build

# Using uv
uv build
```

## Key Implementation Details

### Image Processing Pipeline
- Archives opened with appropriate library based on extension
- Images converted: PIL → QImage (RGB888) → QPixmap
- All images cached in memory as QPixmap objects
- Alphabetical sorting of archive contents

### Zoom and Pan System
- `zoom_factor`: 1.0 = fit to window, >1.0 = zoomed in, minimum 0.2
- `pan_offset`: [x, y] pixel offsets from center
- Keyboard zoom: 1.2x rate for +/- keys
- Mouse wheel zoom: 1.05x rate, centered on cursor position
- `adjust_pan_for_zoom()` keeps zoom centered on specific point
- Pan disabled when zoom_factor == 1.0

### UI Overlay System
- **Main image**: QLabel with black canvas background
- **Help overlay**: Semi-transparent white with HTML formatted text
- **Startup overlay**: Semi-transparent black with loading messages
- Overlays positioned with `raise_()` for proper z-ordering
- Dynamic resizing handled in `resizeEvent()`

### Event Handling Details
- **Keyboard**: 13 different key bindings including WASD pan controls
- **Mouse**: Left button drag for panning (only when zoomed)
- **Wheel**: Zoom in/out with cursor-centered scaling
- **Startup overlay**: Hidden on any key press or mouse click

### Progressive Loading Strategy
1. First image loaded immediately in `load_images()`
2. Remaining images loaded via `QTimer.singleShot(0, self.load_remaining_images)`
3. Startup overlay message updated when loading completes
4. Error handling for `rarfile.BadRarFile` with user-friendly message

### Dependencies and Requirements

- **PyQt5**: Complete GUI framework
- **Pillow**: Image processing (PIL.Image)
- **rarfile**: RAR/CBR archive support
- **External**: `unrar` command-line tool required for RAR files
- **Python**: 3.8+ (defined in pyproject.toml)

### Error Handling Patterns

- Graceful degradation for corrupt/unreadable images
- Comprehensive logging with debug info when verbose mode enabled

### Memory Management

- All images cached as QPixmap objects in `self.images` list
- Images cleared in `closeEvent()` for proper cleanup
- No lazy loading - all archive images loaded into memory
- Canvas pixmap recreated on each `show_image()` call

## Development Notes

- Modular architecture: UI (`viewer.py`), archive processing (`archive_reader.py`), and logging (`logging_setup.py`)
- Test suite with fixtures for all archive formats
- Frameless fullscreen window with manual overlay positioning
- Dual logging system supports both development and user debugging
- Comic book oriented (CBR/CBZ support) but works for any image archives

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CaptSolo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
