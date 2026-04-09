## kinescope-downloader

> CLI utility for downloading DRM-protected videos from Kinescope. Accepts a player JSON config, extracts decryption keys, and downloads video via N_m3u8DL-RE or ffmpeg.

# DRM Media Downloader

## Overview

CLI utility for downloading DRM-protected videos from Kinescope. Accepts a player JSON config, extracts decryption keys, and downloads video via N_m3u8DL-RE or ffmpeg.

## Usage

```bash
# Single video
python3 kinescope_dl.py --json input/player.json --quality 720

# Batch download
python3 kinescope_dl.py --input-dir input --output-dir generated --quality 720
```

## Architecture

```
kinescope_dl.py          → CLI entry point (argparse + sys.exit)
kinescope/
  config.py              → VideoConfig dataclass, parse_config(), sanitize_filename()
  hls.py                 → fetch_m3u8(), parse_master_m3u8(), extract_kid_token(), pick_media_urls(), generate_filtered_m3u8()
  drm.py                 → acquire_key()
  runner.py              → process_single(), run_batch(), select_quality() — orchestrator
  downloaders/
    base.py              → Downloader Protocol
    n_m3u8dl.py           → NM3U8DLDownloader (primary)
    ffmpeg.py            → FFmpegDownloader (fallback, two strategies)
```

### Data Flow

`JSON config` → `config.py` (VideoConfig) → `hls.py` (master m3u8 → media URLs → KID) → `drm.py` (decryption key) → `downloaders/` (download)

Orchestration in `runner.py`: calls modules sequentially, tries N_m3u8DL-RE first, falls back to ffmpeg on failure.

### Layers

- **kinescope_dl.py** — the only place with `sys.exit()`. Catches all module exceptions.
- **Modules** — never call `sys.exit()`. Raise typed exceptions: `ConfigError`, `HLSError`, `DRMError`, `QualityError`, `DownloadError`.
- **Downloaders** — implement the `Downloader` Protocol. New downloader = new class with a `download()` method.

## Conventions

- Python 3.10+, only external dependency is `requests`
- Frozen dataclass for configs (`VideoConfig`)
- Errors are typed exceptions, no `sys.exit()` in library code
- Temporary files in `/tmp/kinescope`, cleaned up after successful download
- `input/` and `generated/` in `.gitignore` — contain user data

## Adding a New Downloader

1. Create a class in `kinescope/downloaders/` with a `download(**kwargs) -> bool` method
2. Add the call in `runner.py:process_single()` in the desired priority order

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yelmuratoff)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yelmuratoff)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
