## socialdownloader

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Flask-based social media video downloader that uses yt-dlp to download videos from YouTube, TikTok, Instagram, Twitter/X, and Facebook. Supports video downloads in multiple resolutions (up to 4K for YouTube) and MP3 audio extraction.

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run the development server (accessible at http://localhost:8080)
python run.py
```

**External dependencies:**
- **FFmpeg** - Required for audio extraction and video re-encoding
- **Deno** - Required for YouTube downloads (since yt-dlp 2025.11.12). Install: `curl -fsSL https://deno.land/install.sh | sh`
- **cookies.txt** (optional) - Place in project root for Twitter/X authenticated downloads

## Architecture

### Core Components

- **`run.py`** - Entry point. Starts Flask app on port 8080 with threading enabled.
- **`app/__init__.py`** - Flask app factory. Initializes shared state (`download_progress` dict, `progress_lock`) and starts the cleanup background thread.
- **`app/routes.py`** - REST API endpoints:
  - `POST /fetch_video_info` - Get video metadata via yt-dlp
  - `POST /start_download` - Start async download, returns `download_id`
  - `GET /download_progress/<id>` - Poll download status
  - `GET /download_file/<id>` - Retrieve completed file
  - Facebook credential management endpoints
- **`app/download_manager.py`** - Contains `download_thread()` which handles the full download pipeline: yt-dlp download → optional FFmpeg re-encoding for video → file renaming.
- **`app/config.py`** - Configuration constants (paths, HTTP headers, cleanup intervals).
- **`app/tasks.py`** - Background cleanup thread that removes downloads older than 2 hours.
- **`app/credentials.py`** - Thread-safe in-memory credential storage for Facebook authentication.
- **`app/utils.py`** - Helper functions for video info extraction and FFmpeg time parsing.

### Download Flow

1. Frontend calls `/fetch_video_info` with URL to get available formats
2. User selects format, frontend calls `/start_download`
3. New thread runs `download_thread()` which updates `download_progress[download_id]`
4. Frontend polls `/download_progress/<id>` for status updates
5. On completion, frontend offers `/download_file/<id>` link

### Threading Model

- Downloads run in daemon threads (one per download)
- `download_progress` dict is protected by `progress_lock` for thread-safe access
- Cleanup runs in a separate daemon thread on 30-minute intervals

### Video Processing

- YouTube: Offers specific resolutions (360p-4K) via format selection
- Other platforms: "Best Available" option using yt-dlp's format selection
- Video files are re-encoded with FFmpeg (libx264/aac) for QuickTime compatibility
- MP3 extraction uses FFmpeg via yt-dlp's postprocessor

---
> Source: [TheMarco/SocialDownloader](https://github.com/TheMarco/SocialDownloader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
