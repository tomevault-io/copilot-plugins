## exif-editor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
This is a Python-based EXIF editor for JPEG images with both CLI and GUI interfaces. The project uses `uv` for package management and provides functionality to edit GPS coordinates, timezone data, and date/time information in image metadata.

## Development Commands

### Package Management
- `uv sync` - Install dependencies (replaces `pip install`)
- `uv add <package>` - Add new dependency
- `uv remove <package>` - Remove dependency

### Running the Applications
- `python edit-exif-cli.py <directory> <latitude> <longitude>` - CLI batch processing
- `python edit-exif-gui.py` - Launch PyQt5 GUI application

## Code Architecture

### Core Module Structure
- `exif.py` - Core EXIF manipulation functions, utilities for GPS/datetime operations
- `edit-exif-cli.py` - Command-line interface for batch processing images
- `edit-exif-gui.py` - PyQt5 GUI application with thumbnail view and metadata editing

### Key Functions in exif.py
- `extract_exif_data()` - Load EXIF data from image files
- `update_image_gps_exif()` - Update GPS coordinates in image metadata
- `update_image_offset_time_exif()` - Update timezone offset data
- `update_local_date_time_by_offset()` - Adjust image timestamps by offset
- `convert_to_dms()` / `convert_from_dms()` - GPS coordinate format conversion

### GUI Architecture (edit-exif-gui.py)
- Main widget uses QHBoxLayout with thumbnail list and sidebar
- Thumbnail list supports multi-selection for batch operations
- Sidebar shows image preview and EXIF metadata when item selected
- Real-time updates of EXIF data after modifications
- Sorting by name, creation time, or DateTimeOriginal

### Dependencies
- `piexif` - EXIF data manipulation
- `pillow` - Image processing
- `pyqt5` - GUI framework (GUI version only)

## Important Implementation Notes
- Uses `piexif.load()` without opening image files for better performance
- GPS coordinates stored in DMS (degrees, minutes, seconds) format in EXIF
- DateTime fields use format: 'YYYY:MM:DD HH:MM:SS'
- Timezone offset format: '[+-]HH:MM'
- Removes thumbnail data from EXIF dictionaries to avoid format compatibility issues

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/allenlee820202)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/allenlee820202)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
