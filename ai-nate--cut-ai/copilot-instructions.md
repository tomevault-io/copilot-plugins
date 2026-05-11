## cut-ai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Cut-AI is a Python CLI tool that automatically extracts highlight clips from long-form video recordings. It uses Gemini 3.0 Pro to analyze VTT transcripts, identify the best 2-3 minute segments, and then clips them from the source video using FFmpeg. It optionally burns in subtitles (monolingual or bilingual English + translated).

## Directory Structure

```
Cut-AI/
  script/          # All Python source code
  docs/            # Reference guides (viral content guide, platform tips)
  data/<session>/  # Source videos and VTT transcripts per recording session
  output/<session>/ # Generated output per session (highlights, clips, transcripts, drafts)
  .claude/skills/  # Claude Code skills (e.g., /viral-clips)
```

## Commands

```bash
# Activate virtualenv (Python 3.14)
source venv/bin/activate

# Step 1: Analyze a VTT transcript to generate highlights.json
python script/cut_ai.py analyze --transcript data/<session>/transcript.vtt --output output/<session>/highlights_<session>.json

# Step 2: Clip highlights from video (plain, no subtitles)
python script/cut_ai.py clip --video data/<session>/video.mp4 --highlights output/<session>/highlights_<session>.json

# Step 3: Clip with burned-in subtitles
python script/cut_ai.py clip --video data/<session>/video.mp4 --highlights output/<session>/highlights_<session>.json --subtitles data/<session>/transcript.vtt

# Step 4: Clip with bilingual subtitles (English + Chinese)
python script/cut_ai.py clip --video data/<session>/video.mp4 --highlights output/<session>/highlights_<session>.json --subtitles data/<session>/transcript.vtt --translate Chinese

# Step 5: Generate dual-language clips (English/ + Chinese/ subdirectories)
python script/cut_ai.py clip_dual --video data/<session>/video.mp4 --highlights output/<session>/highlights_<session>.json --subtitles data/<session>/transcript.vtt --output output/<session>

# Step 6: Extract per-clip transcripts
python script/extract_transcripts.py output/<session>/highlights_<session>.json data/<session>/transcript.vtt --output output/<session>/transcripts

# Full automated pipeline via skill
/viral-clips data/<session>

# Test individual modules directly
python script/vtt_parser.py data/<session>/transcript.vtt
python script/gemini_analyzer.py data/<session>/transcript.vtt
python script/video_clipper.py input.mp4 output.mp4 00:05:00 00:07:30
```

## Architecture

The pipeline is a three-step workflow: **analyze** (AI finds highlights) → **clip** (FFmpeg extracts video) → **draft** (AI writes platform content).

- **`script/cut_ai.py`** — CLI entrypoint with `analyze`, `clip`, and `clip_dual` subcommands. Orchestrates the full pipeline.
- **`script/config.py`** — All configuration: API keys (from `.env`), model name, clip duration constraints, output paths, and the Gemini prompt template.
- **`script/vtt_parser.py`** — Parses WebVTT files into `TranscriptSegment` dataclass objects. Handles speaker detection, BOM removal, timestamp conversion.
- **`script/gemini_analyzer.py`** — `GeminiAnalyzer` class wraps the Google GenAI SDK (`google.genai`) to send the transcript + prompt to Gemini and parse the JSON response into highlight dicts.
- **`script/gemini_translator.py`** — `GeminiTranslator` class for batch translation of subtitle text via Gemini. Uses in-memory caching. Accessed via global singleton `get_translator()`.
- **`script/video_clipper.py`** — FFmpeg wrapper for clipping videos. Two modes: `clip_video()` (fast, lossless `-c copy`) and `clip_video_with_subtitles()` (re-encodes with libx264 to burn in SRT or ASS subtitles). Generates ASS format for bilingual subtitles (styled English + translated text).
- **`script/extract_transcripts.py`** — Extracts per-clip transcript text from highlights.json + VTT file. Saves individual `.txt` files.
- **`docs/viral_content_guide.md`** — Viral content frameworks, archetypes, hook patterns, scoring rubric for clip selection.
- **`docs/platform_content_tips.md`** — Platform-specific formatting rules, character limits, algorithm signals, and tone guidelines for 7 platforms.

## Key Data Flow

1. VTT file → `vtt_parser.parse_vtt()` → `List[TranscriptSegment]`
2. Transcript text → `gemini_analyzer.analyze_transcript()` → highlight JSON (timestamps, titles, categories)
3. Highlights saved to `output/<session>/highlights_<session>.json` for manual review/editing
4. Highlights + video → `video_clipper.clip_video()` or `clip_video_with_subtitles()` → MP4 clips in `output/<session>/`
5. Highlights + VTT → `extract_transcripts.py` → per-clip transcript `.txt` files
6. Transcripts + docs guides → Claude drafts → `output/<session>/video_content_drafts.md`

## Output Structure

```
output/<session>/
  highlights_<session>.json
  English/
    clip_01_Title.mp4
    clip_02_Title.mp4
  Chinese/
    clip_01_Title_zh.mp4
    clip_02_Title_zh.mp4
  transcripts/
    clip_01_Title.txt
    clip_02_Title.txt
  video_content_drafts.md
```

## Environment

- Requires `.env` with `VERTEX_AI_API_KEY` and `GOOGLE_CLOUD_PROJECT`
- Uses Vertex AI mode for Google GenAI SDK (not direct Gemini API keys)
- Model: `gemini-3-pro-preview` (set in `script/config.py`)
- FFmpeg must be installed (`brew install ffmpeg`)

## Conventions

- Highlight categories: `"viral_story"`, `"viral_insight"`, or `"viral_practical"`
- Time format throughout: `HH:MM:SS` (with optional `.mmm` milliseconds in VTT)
- Clips default to 1.5-3 minutes with 0.5s padding on each side
- `highlights_<session>.json` is the intermediate artifact meant to be manually reviewed before clipping
- `data/` holds source videos and VTT transcripts; `output/` holds all generated output per session

---
> Source: [AI-Nate/Cut-AI](https://github.com/AI-Nate/Cut-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
