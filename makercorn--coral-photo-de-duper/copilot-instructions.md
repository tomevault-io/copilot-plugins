## coral-photo-de-duper

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A CLI tool for detecting duplicate images using Google Coral TPU hardware acceleration. The application has dual-mode detection: TPU-accelerated deep learning (via MobileNet V2 embeddings) and perceptual hash fallback when TPU is unavailable.

## Development Environment

### Python Environment
**Use Python 3.11 with venv311** (not venv313 or venv). Python 3.11 is required for TensorFlow Lite with Edge TPU delegate compatibility.

```bash
# Activate environment
source venv311/bin/activate

# Install dependencies
pip install -r requirements.txt
pip install tensorflow-macos  # Required for TPU support
```

### Coral TPU Setup
- **Hardware**: Google Coral USB Accelerator (Vendor ID: 0x1a6e)
- **Runtime**: Edge TPU runtime at `/usr/local/lib/libedgetpu.1.dylib`
- **Model**: MobileNet V2 quantized for Edge TPU (auto-downloads to `models/`)

## Commands

### Running the Application
```bash
# Check TPU status
python dupe_checker.py check-tpu

# Scan for duplicates
python dupe_checker.py scan <path> [--output report.json] [--threshold 0.95]

# Clean up duplicates (always test with --dry-run first)
python dupe_checker.py clean <path> --move-to <dest> --dry-run
python dupe_checker.py clean <path> --move-to <dest> [--keep largest|newest|first]

# Skip TPU (use perceptual hash only)
python dupe_checker.py scan <path> --skip-tpu
```

### Testing
```bash
# Run unit tests
python test_scanner.py

# Run TPU diagnostics
python test_coral.py

# Test with sample images
python dupe_checker.py scan test_images
python dupe_checker.py scan test_images2 --output test_report.json
```

### Code Quality & Linting
```bash
# Format code (black)
python -m black --line-length 100 *.py

# Lint code (ruff)
python -m ruff check *.py
python -m ruff check --fix *.py  # Auto-fix issues

# Type check (mypy)
python -m mypy *.py --ignore-missing-imports

# Run all checks (required before commits)
python -m black --check --line-length 100 *.py && \
python -m ruff check *.py && \
python -m mypy *.py --ignore-missing-imports

# Install linting tools
pip install black ruff mypy
```

**Code Standards**:
- All code must pass black, ruff, and mypy validation
- Line length: 100 characters
- Type hints required on all function signatures
- Use `Optional[T]` for nullable types
- Use `# type: ignore` sparingly with comments explaining why

### Development Utilities
```bash
# View examples
./examples.sh

# Check project structure
ls -la models/  # Downloaded ML models
ls -la test_images*/  # Test image sets
```

## Architecture

### Module Organization

The codebase follows a clean separation of concerns across four main modules:

1. **scanner.py** (`ImageScanner`)
   - Discovers image files in directories
   - Uses `ThreadPoolExecutor` for parallel file I/O
   - Returns list of image metadata dicts: `{'path', 'name', 'size', 'modified', 'created'}`

2. **detector.py** (`DuplicateDetector`)
   - **Dual-mode detection system** (key architectural pattern):
     - **TPU mode**: Loads TensorFlow Lite model with Edge TPU delegate, extracts MobileNet embeddings, computes cosine similarity
     - **Fallback mode**: Uses `imagehash` library (average hash, perceptual hash, difference hash)
   - Multi-phase pipeline:
     - Phase 1: Group by file size (fast filter)
     - Phase 2: SHA256 hash for exact matches
     - Phase 3: Visual similarity via TPU or perceptual hash
   - Returns nested list of duplicate groups: `[[img1, img2], [img3, img4, img5]]`

3. **handler.py** (`DuplicateHandler`)
   - Executes file operations on duplicate groups
   - Keep strategies: `original` (default, smart), `first`, `largest`, `newest`
   - **Smart copy detection**: Identifies filename patterns like `_copy`, `(1)`, `(2)`, etc.
   - Always supports `--dry-run` for safety

4. **dupe_checker.py** (CLI)
   - Click-based CLI with three commands: `scan`, `clean`, `check-tpu`
   - Coordinates the scanner → detector → handler pipeline

### Critical Implementation Details

**TPU Initialization** (detector.py:42-78):
- Must use **full path** to Edge TPU library: `/usr/local/lib/libedgetpu.1.dylib`
- Falls back gracefully to perceptual hash if TPU unavailable
- Model auto-downloads from Google's test_data repo on first use

**Detection Pipeline** (detector.py:91-118):
- Groups images by size first (cheap operation)
- Only compares images within same size group (optimization)
- For small groups (≤10): pairwise comparison
- For large groups: embedding extraction + similarity matrix

**Perceptual Hash Fallback** (detector.py:325-369):
- Computes three hash types: average, perceptual, difference
- Averages similarity scores across all three hashes
- Hash difference ranges 0-64, converted to 0.0-1.0 similarity scale

### Configuration

All thresholds and settings in `config.py`:
- `DEFAULT_SIMILARITY_THRESHOLD = 0.90` (90% match threshold)
- `SUPPORTED_FORMATS` - image extensions to scan (includes `.heic`/`.heif` for Apple formats)
- `IMAGE_SIZE = (224, 224)` - MobileNet V2 input size
- `MAX_WORKERS = 4` - parallel file scanning threads

**Apple HEIC/HEIF Support**: The `pillow-heif` library enables iPhone/iPad photo format support. It's registered at module import in `detector.py` via `register_heif_opener()`.

**RAW Format Support**: The `rawpy` library provides support for 24+ RAW formats from all major camera manufacturers (Canon, Nikon, Sony, Fujifilm, etc.). RAW files are automatically detected by extension and converted to RGB via LibRaw postprocessing in `_load_image()` method.

**Smart Copy Detection** (handler.py): The `original` strategy (default) intelligently detects copy filename patterns (`_copy`, `(1)`, `(2)`, `duplicate`, etc.) via regex matching in `_is_copy_filename()`. Files are ranked by priority (0=original, 100+=copy), with originals always preserved. Uses multi-level sorting: (is_copy, priority, -size) to handle edge cases.

### Data Flow

```
User Input (paths)
    ↓
ImageScanner.scan() → List[Dict] of image metadata
    ↓
DuplicateDetector.find_duplicates() → List[List[Dict]] of duplicate groups
    ↓
DuplicateHandler.move_duplicates() or .remove_duplicates()
```

Each dict contains: `{path: Path, name: str, size: int, modified: float, created: float}`

## Development Notes

### When Modifying Detection Logic
- The `use_tpu` flag controls whether TPU is attempted
- The `threshold` parameter (0.0-1.0) controls similarity strictness
- Both TPU and perceptual hash modes must respect the same threshold
- Always test with both `--skip-tpu` and TPU-enabled modes

### When Adding New Features
- Follow the existing separation: scanner handles I/O, detector handles comparison, handler handles file ops
- CLI validation in `dupe_checker.py`, core logic in respective modules
- Use progress bars (`tqdm`) for long-running operations

### Edge TPU Delegate Loading
If TPU initialization fails with library not found:
- Check `/usr/local/lib/libedgetpu.1.dylib` exists
- Verify Coral device is connected via `system_profiler SPUSBDataType | grep -i coral`
- The hardcoded path in `detector.py:54` is macOS-specific; Linux uses `/usr/lib/libedgetpu.so.1`

### Testing Strategy
- `test_images/`: Pattern-based images for visual duplicate testing
- `test_images2/`: Photo-like images with exact and near-duplicates
- `test_scanner.py`: Unit tests for scanner, detector, handler modules
- `test_coral.py`: Hardware diagnostics for TPU detection

## Threshold Tuning Guidelines

- **0.99**: Only near-identical images (strict)
- **0.95**: Default, catches most duplicates
- **0.90**: More lenient, catches similar images
- **0.85**: Very lenient, may catch visually similar but not duplicate images

For perceptual hash mode, lower thresholds (0.85-0.90) work well since hash similarity is more binary than embedding cosine similarity.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MakerCorn)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MakerCorn)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
