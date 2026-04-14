## audio-processor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an audio processing tool that splits monologue audio recordings (containing speech and/or music) into chunks based on silence detection. The core algorithm detects 25-second silence gaps and splits audio files at these boundaries.

**Critical Silence Detection Rule**: When 25 seconds of silence is detected, the chunk boundary is placed at the START of the silence period (end of previous chunk) and the next chunk begins at the END of the silence period. This preserves the 25-second silence gap between chunks.

## Core Architecture

- `audio_splitter.py`: Main processing logic with silence detection using PyDub
- `run.py`: Convenience entry point that imports and runs the main function
- `main.py`: Basic entry point (currently minimal)
- `input/`: Directory for source audio files
- `output/`: Directory for split audio chunks, organized by source filename

## Key Processing Parameters

- **min_silence_len**: 25000ms (25 seconds) - threshold for chunk boundaries
- **silence_thresh**: -40dB - amplitude threshold for silence detection
- **keep_silence**: 1000ms (1 second) - silence preserved at chunk boundaries
- **min_chunk_duration**: 5000ms (5 seconds) - chunks shorter than this are discarded
- **output_format**: FLAC files optimized for speech transcription (16kHz, 16-bit, mono)

## Development Commands

```bash
# Install dependencies
uv sync

# Main CLI with Typer (recommended)
uv run python audio_splitter_cli.py --help
uv run python audio_splitter_cli.py process --interactive
uv run python audio_splitter_cli.py optimize input.wav --output optimized.flac

# Alternative interfaces
uv run python run.py                    # Simple batch processor
uv run python main.py                   # Basic entry point
```

## Supported Audio Formats

Input: MP3, WAV, M4A, FLAC, OGG, AAC, MP4
Output: FLAC format (optimized for speech transcription)

## Dependencies

### Core Processing
- **Librosa**: Advanced audio analysis and processing engine
- **SoundFile**: Audio I/O operations
- **NumPy**: Numerical operations for audio data
- **FFmpeg**: System dependency for optimal FLAC conversion

### CLI and User Interface
- **Typer**: Modern CLI framework with rich features
- **Rich**: Beautiful terminal output and progress bars
- **Pydantic**: Configuration validation and type safety
- **PyYAML**: Configuration file support

### Monitoring and Progress
- **tqdm**: Progress bars for batch processing
- **psutil**: Memory usage monitoring

## Processing Workflow

1. **File Discovery**: Scan input directory for supported audio formats (excluding `input/processed/`)
2. **Audio Loading**: Load files using Librosa with timing and size reporting
3. **Silence Analysis**: Detect silence gaps using advanced signal processing
4. **Chunk Filtering**: Remove chunks shorter than 5 seconds
5. **FLAC Optimization**: Convert using FFmpeg to 16kHz, 16-bit, mono FLAC
6. **Export**: Save optimized files with metadata in organized subdirectories
7. **File Management**: Move successfully processed files to `input/processed/` (optional)

## Output Structure

```
output/
├── audio_file1/
│   ├── metadata/
│   │   ├── processing_metadata.json
│   │   ├── chunks.csv
│   │   └── processing_report.txt
│   ├── audio_file1_full_16k.flac      # Complete optimized file
│   ├── audio_file1_chunk01_16k.flac   # Individual chunks
│   └── audio_file1_chunk02_16k.flac
└── audio_file2/
    ├── metadata/
    ├── audio_file2_full_16k.flac
    └── ...
```

## Enhanced CLI Features

### Modern Interface with Typer
The main CLI (`audio_splitter_cli.py`) provides:

**Interactive Configuration**:
- Step-by-step parameter setup with explanations
- FLAC compression level selection (0-12)
- Sample rate optimization for speech transcription
- Visual configuration tables with Rich formatting

**Subcommands**:
- `process`: Main audio processing with FLAC optimization
- `config`: Configuration management (show/create/edit)
- `optimize`: Single file quick optimization
- `analyze`: Audio file analysis without processing

**User Experience**:
- Beautiful Rich formatting with colors and progress bars
- Type-safe configuration with Pydantic validation
- Dry-run mode with size reduction predictions
- Comprehensive help and usage examples

### FLAC Optimization for Speech Transcription
- **16kHz sample rate**: Optimal for speech intelligibility and LLM processing
- **Mono conversion**: 50% size reduction without quality loss
- **16-bit depth**: Quality preservation while minimizing file size
- **Compression level 8**: Excellent compression without encoding overhead
- **Size reduction**: Typical 90-95% reduction from original recordings

### Configuration Management
- YAML and JSON configuration file support
- Interactive configuration with validation
- Command-line parameter overrides
- Clean configuration output without Python objects

## Important Processing Details

### Chunk Count Reporting
The session summary reports the number of audio chunks created, **excluding** the full optimized file. This provides an accurate count of the actual splits created from the original audio.

### File Management
- **Automatic Organization**: Successfully processed files are moved to `input/processed/` by default
- **Clean Input Directory**: Subsequent runs automatically skip files in the `processed` subdirectory
- **Disable Option**: Use `--no-move` flag to keep original files in place
- **Safe Operation**: Files are only moved after successful processing to prevent data loss

## Git Strategy

When committing changes:
- Do not include Claude signature or "Generated with Claude Code" attribution
- Write concise, descriptive commit messages that explain the work done
- Focus on what was changed and why, keeping messages reasonably brief
- Follow the project's existing commit message style and conventions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alikebrahim)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alikebrahim)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
