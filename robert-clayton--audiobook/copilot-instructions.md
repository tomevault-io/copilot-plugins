## audiobook

> Automated pipeline for converting web novel chapters into audiobooks using Qwen3 TTS Base with multi-speaker voice cloning and audio effects. Supports scraping from RoyalRoad and ScribbleHub, as well as locally-managed chapter files (e.g. translated novels).

# CLAUDE.md

## Project Overview

Automated pipeline for converting web novel chapters into audiobooks using Qwen3 TTS Base with multi-speaker voice cloning and audio effects. Supports scraping from RoyalRoad and ScribbleHub, as well as locally-managed chapter files (e.g. translated novels).

Runs as a NiceGUI web dashboard (default) or headless CLI. State is tracked in SQLite for chapter-level status, retry, and filesystem reconciliation.

**Actively used in production** — changes must be careful and non-breaking.

## Tech Stack

- **Language:** Python 3.11 (strict: >=3.11, <3.12)
- **Dependency Manager:** uv
- **TTS Engine:** Qwen3 TTS Base (default, voice cloning, CUDA-accelerated); optional Coqui TTS (XTTS v2)
- **Web GUI:** NiceGUI 2.x (Quasar/Vue3, dark industrial theme)
- **Audio Processing:** FFmpeg (external dependency, must be on PATH)
- **ML:** PyTorch 2.5.1 + CUDA 12.1, Transformers >=4.57
- **Scraping:** BeautifulSoup4, requests, cloudscraper (CloudFlare bypass)
- **Text Processing:** NLTK (sentence tokenization)
- **State:** SQLite via ChapterDB (series + chapters tables)

## Commands

```bash
# Install dependencies
uv sync

# Launch web dashboard (default, http://localhost:8080)
uv run audiobook [--dev]

# Run headless CLI pipeline (scrape + generate)
uv run audiobook --cli [--dev]

# --dev: use config_dev.yml instead of config.yml
#        Both loading AND saving use the same file, so --dev
#        keeps production config.yml untouched.
```

### Coqui TTS Setup (separate venv)

Coqui TTS requires `transformers<4.41`, which conflicts with Qwen3 TTS (`transformers>=4.57`).
To use the Coqui engine, create a separate venv:

```bash
uv venv .venv-coqui --python 3.11
# Linux/macOS:
source .venv-coqui/bin/activate
# Windows:
.venv-coqui\Scripts\activate

uv pip install coqui-tts nltk bs4 cloudscraper python-dotenv
uv pip install --reinstall torch torchaudio --index-url https://download.pytorch.org/whl/cu121
uv pip install -e . --no-deps       # audiobook entry point only
# Set tts_engine: coqui in config.yml, then:
python -m audiobook --cli
```

### Adding New Speakers

Extract a reference audio clip and generate a transcript for voice cloning.

**From an audiobook (.m4b) on the network share:**
```bash
ffmpeg -ss 300 -t 20 -i "//10.0.0.2/media/audiobooks/path/to/book.m4b" -ar 24000 -ac 1 -y speakers/<name>.wav
whisper speakers/<name>.wav --model base --output_format txt --output_dir speakers/
```

**From a YouTube video/short:**
```bash
yt-dlp -x --audio-format wav -o speakers/<name>.wav "<youtube_url>"
ffmpeg -ss 0 -t 6 -i speakers/<name>.wav -ar 24000 -ac 1 -y speakers/<name>_trimmed.wav
mv speakers/<name>_trimmed.wav speakers/<name>.wav

# (Optional) Isolate vocals if clip has background music/noise
uv run --with soundfile demucs --mp3 -n htdemucs --two-stems vocals speakers/<name>.wav

whisper speakers/<name>.wav --model base --output_format txt --output_dir speakers/
```

**Guidelines:**
- 10-60 seconds of clean speech is ideal
- 24kHz mono WAV format required (`-ar 24000 -ac 1`)
- Speaker name must match narrator/mapping values in config (without extension)
- The `.txt` transcript significantly improves voice cloning quality; without it, falls back to x-vector-only mode

## Architecture

```
audiobook/
├── __main__.py          # Entry point -> cli.main()
├── cli.py               # Arg parsing, launches GUI or headless CLI
├── config.py            # YAML config loader/saver
├── pipeline.py          # All pipeline phases: scrape, audio, single-series/chapter ops
├── state.py             # ChapterDB: SQLite state tracking (series + chapters)
├── scrapers/
│   ├── base.py          # Abstract BaseScraper, 100+ anti-scrape filters, ChapterUnavailableError
│   ├── royalroad.py     # RoyalRoad scraper with system message detection
│   └── scribblehub.py   # ScribbleHub scraper with CloudFlare bypass
├── processors/
│   ├── processing.py    # Orchestrates TTS pipeline per series/chapter
│   ├── tts_processor.py # Core TTS: chunking, speaker tags, garbled detection, audio merging
│   ├── tts_instance.py  # Singleton Coqui TTS model (GPU, optional)
│   └── tts_qwen.py      # Singleton Qwen3 TTS model (GPU, default)
├── validators/
│   └── validate_file.py # Text cleaning: encoding fixes, acronyms, replacements
├── utils/
│   ├── audio.py         # FFmpeg wrappers: merge, modulate, speed, mp3 convert
│   └── colors.py        # ANSI terminal color codes
└── web/
    ├── app.py           # NiceGUI app setup, FastAPI audio route, page routing
    ├── dashboard.py     # Main dashboard: series table, pipeline controls, live log
    ├── series_page.py   # Series detail: chapter list, rescrape, regenerate, filename fixes
    ├── runner.py        # PipelineRunner: background thread execution, state management
    ├── shared.py        # Shared UI helpers (status badges, diff rendering, table updates)
    ├── theme.py         # Dark industrial theme (colors, CSS, JetBrains Mono)
    └── log_capture.py   # Thread-aware stdout/stderr capture for GUI log panel
```

## Key Concepts

- **Two-phase pipeline:** Phase 1 scrapes new chapters (skips local series). Phase 2 syncs filesystem, then processes text -> TTS -> WAV -> speed adjust -> MP3.
- **Local series:** Set `url: local` in config for manually-managed chapter files (e.g. translations). No scraping occurs; drop `.txt` files into `{output_dir}/{name}/raws/` and the audio phase picks them up via `sync_filesystem`.
- **Speaker tags:** `<<SPEAKER=name>>...<</SPEAKER>>` tags in text map characters to voice profiles in `speakers/`.
- **System voice:** Certain HTML elements (bold, italic, tables, etc.) get wrapped as "system" speaker with modulation effects (flanger + chorus).
- **Anti-scrape filtering:** `base.py` maintains 100+ hardcoded anti-piracy messages to strip from scraped content, including embedded removal within larger text blocks.
- **Config-driven:** `config.yml` defines series with URL, narrator, replacements, system message settings, and character-to-voice mappings.
- **Singleton TTS:** `QwenTTSInstance` (default) or `TTSInstance` (Coqui) loads the GPU model once, shared across all processing. Unloaded after each pipeline run.
- **Speaker transcripts:** `speakers/*.txt` files contain reference audio transcripts for Qwen3 voice cloning quality. Missing `.txt` falls back to x-vector-only mode (lower quality).
- **Garbled audio detection:** Chunks exceeding 100s duration are likely TTS hallucinations. Retried up to 2 times, then the chapter is marked failed.
- **ChapterDB:** SQLite database at `{output_dir}/audiobook.db` tracks chapter status (pending -> processing -> done/failed), raw/output paths, source URLs, retry counts.

## Data Flow

```
config.yml -> scrape chapters (or manually place in raws/) -> save .txt to {output_dir}/{series}/raws/
  -> sync_filesystem (register new files, reconcile DB state)
  -> validate/clean text -> split into chunks (750 chars Qwen / 250 chars Coqui)
  -> TTS per chunk (batched, 5 at a time) -> modulate system voice -> adjust narrator volume
  -> merge chunks -> convert to MP3
  -> mark done in DB
```

## Web GUI

The default launch mode opens a NiceGUI dashboard at `http://localhost:8080`.

**Dashboard (`/`):**
- Series summary table (done/pending/failed counts per series)
- Pipeline controls: Run Full Pipeline, Scrape Only, Sync Filesystem
- Live log panel with thread-aware capture
- Pipeline state indicator (Idle/Scraping/Generating/Finished/Error)

**Series page (`/series/{name}`):**
- Chapter table with status, published date, errors
- Per-series actions: Scrape, Generate, Rescrape All (with diff review dialog), Fix Filenames
- Per-chapter actions: Play audio, Regenerate, Rescrape (with diff preview)
- Audio playback via `/api/audio/{chapter_id}`

**Runner:** `PipelineRunner` manages all operations in background daemon threads with:
- Config reload/save lifecycle
- DB connection management
- TTS model unload on completion
- Crash recovery (reset stale "processing" chapters on shutdown)

## Configuration (config.yml)

```yaml
config:
  output_dir: //10.0.0.2/media/audiobooks/Generated
  tts_engine: qwen          # or "coqui"
  narrators:                 # per-narrator settings (pause, volume)
    default:
      pause: 0.3             # default silence padding for all narrators
    jareth:
      pause: 0.3
      volume: 1.3            # loudness correction (1.0 = no change)
    katie:
      pause: 0.2

series:
  # Web novel (scraped)
  - name: Series Name
    url: https://www.royalroad.com/fiction/12345/series-name
    narrator: travis_baldree
    latest: https://...       # auto-updated by scraper
    enabled: true             # default true
    replacements:
      Mana: mah-nah
    system:
      voice: onyx
      modulate: true
      speed: 1.0
      type: [bold, italic, bracket, angle, blockquote, table, center]
    mappings:
      Character A: speaker_one
      Character B: speaker_two

  # Local series (manually managed)
  - name: "That's It. Let's Turn Slaves into Adventurers"
    url: local
    narrator: some_speaker
```

## Development Notes

- **No CI/CD, no linter config, no active tests.** The `tests/` dir exists but is empty.
- `dev/` directory contains experimental code (image gen, LLM tagging) and is gitignored.
- `config*.yml` files are gitignored — they contain user-specific series lists and network paths.
- `.env` holds `HUGGINGFACE_TOKEN` — never commit this.
- Output goes to a network share path configured in `config.yml`.

## Commit Convention

Use conventional commit prefixes: `feat:`, `fix:`, `refactor:`, `enhance:`, `bugfix:`, `docs:`, `chore:`

## Important Warnings

- **config.yml is live state** — the `latest` field is auto-updated by the scraper to track progress. Do not reset or alter `latest` values without understanding the consequences.
- **Singleton TTS model** — changes to `tts_qwen.py` or `tts_instance.py` affect all audio generation globally.
- **Anti-scrape list in base.py** — these strings must be exact matches of messages found on source sites. Do not reformat or deduplicate without verifying.
- **FFmpeg commands** — audio utils shell out to ffmpeg. Test changes with actual audio files.
- **Speaker files** — voice profiles in `speakers/` are WAV files used for voice cloning. Names must match narrator/mapping values in config (without extension).
- **ChapterDB is source of truth for processing state** — `sync_filesystem` reconciles DB with disk (registers new files, reverts missing outputs, cleans orphaned entries). The DB lives at `{output_dir}/audiobook.db`.
- **GUI runs pipeline in daemon threads** — `PipelineRunner` methods are not thread-safe for concurrent calls; the GUI enforces single-run via `is_running` checks.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/robert-clayton)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/robert-clayton)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
