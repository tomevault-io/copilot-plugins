## bhajan

> > Context and findings for future AI agents working on this codebase.

# AI Agent Notes for bhajan

> Context and findings for future AI agents working on this codebase.

## Project Overview

`bhajan` is a cross-platform CLI tool that generates karaoke videos from YouTube URLs. It downloads audio, separates vocals/instrumental, transcribes lyrics, and either renders a video or launches a GUI player with synchronized lyrics.

## Key Architecture Decisions

### Pipeline Stages
1. **Download** (`stages/download.py`) - yt-dlp, auto-cleans YouTube tracking params
2. **Normalize** (`stages/normalize.py`) - ffmpeg EBU R128 loudnorm
3. **Separate** (`stages/separator.py`) - pluggable: Demucs or audio-separator
4. **Transcribe** (`stages/transcription.py`) - pluggable: faster-whisper or Parakeet
5. **Subtitles** (`stages/subtitles.py`) - ASS (karaoke style) + LRC formats
6. **Render/GUI** - Either ffmpeg video render OR tkinter GUI player

### Critical Files

| File | Purpose | Key Details |
|------|---------|-------------|
| `src/bhajan/pipeline.py` | Main orchestration | `KaraokePipeline.run()` executes all stages |
| `src/bhajan/cli.py` | CLI entry point | Uses `click`, defines all flags |
| `src/bhajan/config.py` | Constants & defaults | Font sizes, colors, model defaults |
| `src/bhajan/utils.py` | Utilities | `safe_filename()`, `clean_youtube_url()`, `parse_artist_and_title()` |
| `src/bhajan/gui_player.py` | GUI karaoke player | tkinter + pygame, word-level highlighting |

## Important Implementation Details

### YouTube URL Cleaning
**Function:** `utils.clean_youtube_url()`

YouTube URLs often have tracking parameters that cause issues:
- Input: `https://youtube.com/watch?v=ABC&list=...&start_radio=...`
- Output: `https://youtube.com/watch?v=ABC`

The pipeline calls this automatically at the start of `run()`.

### Safe Filename Handling
**Function:** `utils.safe_filename()`

Windows has strict filename rules:
- Cannot end with space or period
- Cannot contain `<>?:"/\|*`
- Max path length issues

The function:
1. Replaces invalid chars with `_`
2. Collapses multiple `_` or spaces
3. Strips leading/trailing `_` and spaces
4. Truncates to max_len (default 80)
5. Strips again after truncation (critical!)

**Bug history:** Previously left trailing spaces when truncating at word boundary, causing `[WinError 3] The system cannot find the path specified`.

### Metadata Extraction
**Functions:** `utils.clean_track_name()`, `utils.parse_artist_and_title()`

YouTube titles often have prefixes that break lyrics search:
- "Mix - Artist - Track" → extract "Artist - Track"
- "Artist - Topic" → handle specially
- "Artist 'Track' Official Video" → parse artist from quotes

These are used when `fetch_lyrics=True` to improve LRCLib API results.

### Pluggable Backends

Both separation and transcription use a registry pattern:

```python
# stages/separator.py
from bhajan.stages.separator_base import SeparatorBackend, SeparationResult

class MySeparator(SeparatorBackend):
    def name(self) -> str: return "my_backend"
    def available(self) -> bool: return True  # Check deps
    def separate(self, audio_path: Path, output_dir: Path) -> SeparationResult: ...

register(MySeparator, "my_backend")
```

Same pattern for transcription in `stages/transcription.py`.

### CUDA Handling

The codebase has multiple CUDA fallback strategies:

1. **Demucs separator** (`stages/separator_demucs.py`):
   - Tries `python -m demucs --device cuda`
   - On `CalledProcessError`, retries with `--device cpu`
   - Logs: "CUDA separation failed, falling back to CPU..."

2. **Audio-separator backend** (`stages/separator_audio_separator.py`):
   - Uses `audio_separator.Separator()` with `device="cuda"` or `"cpu"`
   - GPU acceleration requires `audio-separator[gpu]` install

3. **Whisper transcription** (`stages/transcription_whisper.py`):
   - Uses `faster-whisper` with `device="cuda"` or `"cpu"`
   - Auto-detects CUDA availability

4. **Parakeet transcription** (`stages/transcription_parakeet.py`):
   - NVIDIA NeMo toolkit, CUDA only
   - Requires: `pip install nemo_toolkit['asr']`

### GUI Player

**File:** `src/bhajan/gui_player.py`

- Uses **tkinter** for UI (built into Python)
- Uses **pygame** for audio playback (added to deps)
- Features:
  - Word-by-word highlighting synchronized to audio
  - Progress bar with seek
  - Space = play/pause, arrows = seek
  - Large 32px font for lyrics

**Launch:** `bhajan "URL" --gui`

The GUI skips video rendering entirely - much faster than ffmpeg encoding.

### Subtitle Rendering

**ASS Style Configuration** (`config.py`, `stages/subtitles.py`):

Key settings for readable karaoke:
```python
DEFAULT_FONT_SIZE = 96  # Large!
DEFAULT_OUTLINE_WIDTH = 4
DEFAULT_OUTLINE_COLOR = "&H00FFFFFF"  # White outline
```

**Alignment:** 5 = center-middle (was 2 = bottom-center)
**Margins:** Reduced from 250 to 50 pixels

The ffmpeg filter in `stages/render.py` forces these styles:
```python
force_style='FontSize=96,Alignment=5,...'
```

### Intermediate File Cleanup

Default changed from `keep_intermediate=True` to `False`:
- Without `--keep-intermediate`: cleans up source/, stems/, transcript/, subtitles/
- With `--keep-intermediate`: retains all for debugging

Cleanup happens at end of `pipeline.run()` if not skipped.

## Common Issues & Solutions

### Issue: "Separator backend 'audio-separator' is not available"
**Cause:** `audio-separator` package not installed
**Fix:** `pip install audio-separator` or `uv add audio-separator`

### Issue: "This video is not available" (yt-dlp)
**Cause:** YouTube URL has playlist params that redirect to unavailable video
**Fix:** URL auto-cleaning should strip `&list=...` etc. Already implemented.

### Issue: "[WinError 3] The system cannot find the path specified"
**Cause:** Safe filename left trailing space after truncation
**Fix:** Applied in `utils.py` - strip after truncation

### Issue: Demucs CUDA fails silently or errors
**Cause:** CUDA not available or PyTorch CUDA not installed
**Fix:** Auto-fallback to CPU implemented in `separator_demucs.py`

### Issue: "main() got an unexpected keyword argument 'gui'"
**Cause:** CLI `main()` function signature doesn't match `click` options
**Fix:** Ensure `gui: bool` parameter in `cli.py main()` function

### Issue: Parakeet backend not available
**Cause:** `nemo_toolkit` not installed
**Fix:** Optional dependency - install with `pip install nemo_toolkit['asr']`

### Issue: GUI player shows but no audio
**Cause:** pygame not installed or audio driver issues
**Fix:** `pygame` is in main deps now. On Linux may need `alsa` or `pulse`.

## Dependencies

### Core (in pyproject.toml)
```toml
dependencies = [
    "yt-dlp>=2024.0.0",
    "faster-whisper>=1.0.0",
    "demucs<4.1",
    "rich>=13.0.0",
    "click>=8.1.0",
    "torchaudio==2.5.1",
    "soundfile>=0.13.1",
    "torchcodec>=0.11.0",
    "requests>=2.31.0",
    "pygame>=2.5.0",  # Added for GUI
]
```

### Optional
- `nemo_toolkit['asr']` - Parakeet transcription (NVIDIA GPU)
- `audio-separator[gpu]` - Better separation with GPU support

## Testing Commands

Quick validation:
```bash
# Test URL cleaning
uv run python -c "from bhajan.utils import clean_youtube_url; print(clean_youtube_url('https://youtube.com/watch?v=ABC&list=XYZ'))"

# Test safe filename
uv run python -c "from bhajan.utils import safe_filename; print(repr(safe_filename('Test Name ', 10)))"

# Test pipeline (skip render for speed)
uv run bhajan "https://youtube.com/watch?v=aqz-KE-bpKQ" --skip-render -v

# Test GUI mode
uv run bhajan "https://youtube.com/watch?v=aqz-KE-bpKQ" --gui
```

## File Structure

```
src/bhajan/
├── __init__.py
├── cli.py                    # CLI entry point
├── config.py                 # Constants
├── gui_player.py             # GUI karaoke player
├── logger.py                 # Rich logging setup
├── pipeline.py               # Main orchestration
├── subprocess_utils.py       # Subprocess helpers
├── utils.py                  # safe_filename, URL cleaning, metadata parsing
└── stages/
    ├── download.py
    ├── normalize.py
    ├── render.py
    ├── lyrics_fetch.py       # LRCLib integration
    ├── subtitles.py          # ASS/LRC generation
    ├── separator.py          # Backend registry
    ├── separator_base.py     # Abstract base class
    ├── separator_demucs.py   # Demucs backend
    ├── separator_audio_separator.py  # audio-separator backend
    ├── transcription.py      # Backend registry
    ├── transcription_base.py # Abstract base + data models
    ├── transcription_whisper.py      # faster-whisper backend
    └── transcription_parakeet.py     # Parakeet backend
```

## Important Notes for Future Agents

1. **Always use `uv run`** instead of direct Python - ensures deps are synced
2. **YouTube URLs need cleaning** - always use `clean_youtube_url()` before download
3. **Safe filenames must strip spaces** - Windows hates trailing spaces
4. **CUDA fallback is essential** - many users have NVIDIA but broken CUDA setup
5. **GUI mode is preferred** - video rendering is slow, GUI is fast
6. **Default cleanup saves disk** - intermediate files are large (WAV stems)
7. **Metadata extraction improves lyrics** - clean "Mix -" prefixes for LRCLib

## Last Updated

2025-01-09 - Added GUI mode, Parakeet backend, audio-separator backend, URL cleaning, CUDA fallback, improved subtitle rendering.

---
> Source: [rithamnatani/bhajan](https://github.com/rithamnatani/bhajan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
