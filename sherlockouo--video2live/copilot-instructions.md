## video2live

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

### Release Management (New)
- **Check dependencies**: `python release.py --check-only`
- **Build only**: `python release.py --build-only`
- **Full release with patch bump**: `python release.py --bump patch`
- **Full release with minor bump**: `python release.py --bump minor`
- **Dry run build**: `python release.py --bump patch --dry-run`
- **Custom version**: `python release.py --version 1.2.3`
- **Skip upload**: `python release.py --bump patch --skip-upload`
- **Skip git tag**: `python release.py --bump patch --skip-tag`
- **TestPyPI upload**: `python release.py --bump patch --repository testpypi`

### Legacy Build Commands
- **Build package**: `python setup.py sdist bdist_wheel`
- **Publish to PyPI**: `twine upload dist/*` (via build.sh script)
- **Clean build artifacts**: `rm -rf dist build video2live.egg-info`
- **Install locally**: `pip install -U video2live`
- **Install development**: `pip install -e .`

### Testing and Usage
- **Run tool**: `video2live ./input.mp4 output_dir`
- **Run without importing to Photos**: `video2live ./input.mp4 output_dir --no-import`
- **Test locally**: `python -m video2live ./input.mp4 output_dir`

### Time-based Video Extraction (New Feature)
- **Extract middle portion**: `video2live ./input.mp4 output_dir -ss 5 -to 8 -c 5.5`
- **Extract from specific time**: `video2live ./input.mp4 output_dir -ss 2 -d 3`
- **Custom cover from specific time**: `video2live ./input.mp4 output_dir -ss 1 -to 4 -c 2.5`

## Development Workflow

### Standard Release Process
1. **Feature development**: Work on new features/bug fixes
2. **Testing**: Ensure all functionality works correctly (CLI, time extraction, etc.)
3. **Release**: Use `python release.py --bump patch/minor/major`
   - Automatically updates version in `setup.py`
   - Builds both source distribution and wheel
   - Uploads to PyPI
   - Creates git tag
4. **Alternative release options**:
   - Dry run: `python release.py --bump patch --dry-run`
   - Build only: `python release.py --build-only`
   - Version bump without upload: `python release.py --bump patch --skip-upload`

### Scripts Available
- `release.py`: Comprehensive release management with versioning, builds, uploads, and git tagging
- `build.sh`: Legacy simple build script (deprecated in favor of release.py)
- `setup.py`: Package configuration

## Code Architecture

This is a Python command-line tool that converts videos to iOS Live Photos. The architecture follows a simple modular design:

### Core Components
- **video2live/__main__.py**: Entry point that imports and calls main() from cli.py
- **video2live/cli.py**: Main CLI implementation containing all core functionality
- **setup.py**: Package configuration with dependencies and entry point

### Key Dependencies
- **moviepy**: Video processing and frame extraction
- **makelive**: Live Photo metadata creation
- **ffmpeg**: System dependency for video format conversion
- **imageio/ Pillow**: Image processing

### Workflow Architecture
1. **Input Validation**: Check video file existence and system dependencies, validate time parameters (`-ss` for start time, `-to` for end time, ensure `-to` > `-ss`)
2. **Parameter Processing**: Parse time extraction parameters similar to ffmpeg design:
- `-ss time`: start time in seconds
- `-to time`: end time in seconds (implies duration = to - ss)
- `-c time`: custom cover frame time (defaults to start of extracted segment)
- `-d duration`: Live Photo duration with time-slice logic
3. **Frame Extraction**: Extract frame at specified time for Live Photo cover (supports custom cover time)
4. **Time-based Video Extraction**: Use ffmpeg with time-seeking to extract specific video segment
5. **Video Conversion**: Convert to H.264/AAC MOV format with proper time bounds, using ffmpeg's time-based filtering (`-ss` before `-i`, `-to` after `-i`)
6. **Live Photo Creation**: Use makelive library to combine JPG and MOV with proper metadata
7. **Photo Import**: Optional AppleScript integration to import into Photos app (macOS only)

### Error Handling
The tool validates system dependencies (ffmpeg, makelive) before processing and provides descriptive error messages. All subprocess operations include proper error checking.

### Platform Requirements
- macOS only for Photos app integration
- Python 3.8+ required
- ffmpeg system dependency must be installed

---
> Source: [Sherlockouo/video2live](https://github.com/Sherlockouo/video2live) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
