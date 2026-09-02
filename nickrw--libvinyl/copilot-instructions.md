## libvinyl

> libvinyl is a CLI tool for organising vinyl recordings from a Teenage Engineering

# Agent Instructions

## Project overview

libvinyl is a CLI tool for organising vinyl recordings from a Teenage Engineering
TP-7. It splits multi-track WAV files into individual tracks, looks up metadata
from MusicBrainz, converts to FLAC, and manages a library of albums.

## Architecture

- `src/libvinyl/cli.py` — Click CLI entry point and pipeline orchestration
- `src/libvinyl/audio.py` — Audio analysis (RMS energy, silence detection,
  duration-first splitting) and WAV file manipulation
- `src/libvinyl/musicbrainz.py` — MusicBrainz API integration
- `src/libvinyl/convert.py` — WAV→FLAC conversion (via ffmpeg) and metadata
  tagging (via mutagen)
- `src/libvinyl/state.py` — YAML-based state management with per-album status
  tracking (`raw → analyzed → split → converted → done`)
- `src/libvinyl/ui.py` — Rich console output (tables, prompts, progress bars)

## Key design decisions

- **Duration-first splitting**: When MusicBrainz data is available, we predict
  track boundaries from known durations and search for the quietest audio region
  nearby, rather than trying to detect all silences globally. Vinyl has surface
  noise that makes global silence detection unreliable.
- **State machine**: Each album progresses through states and can resume from
  any point after interruption. State is stored in `library-state.yaml` at the
  library root.
- **Interactive**: The tool always shows a preview and asks for confirmation
  before making changes. It prompts for MusicBrainz release selection and
  offers manual fallback.

## Conventions

- Python 3.12+, managed with uv
- Type hints throughout, `from __future__ import annotations`
- Dependencies: click, rich, pydub, scipy, numpy, musicbrainzngs, mutagen, pyyaml
- CLI entry point: `libvinyl` (defined in pyproject.toml)
- Use `ui.py` functions for all user-facing output (don't print directly)

## Testing

No test suite yet. To verify changes manually:

```bash
uv run libvinyl status ./library
uv run libvinyl process ./library --album "Artist - Album"
```

The `library/` directory contains real WAV files — be careful with destructive
operations. The `archive/` directory (sibling to `library/`) stores original
WAVs after processing.

---
> Source: [nickrw/libvinyl](https://github.com/nickrw/libvinyl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
