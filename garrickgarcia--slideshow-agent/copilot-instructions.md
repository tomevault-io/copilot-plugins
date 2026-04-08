## slideshow-agent

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SlideShow-Agent is an AI agent that creates video presentations with AI-generated slides, voiceovers, and animated transitions. It uses fal.ai APIs and FFmpeg for video assembly.

**Workflow**: User prompt -> Generate slides (Nano Banana Pro) -> Generate voiceovers (Eleven Labs) -> Generate transitions (Kling 2.6 Pro) -> Assemble video (FFmpeg) -> Output MP4

## Skill Location

The main skill is at `.claude/skills/slideshow-video-agent/SKILL.md`

## Python Environment

- **Virtual environment**: `.venv\Scripts\python.exe` (Python 3.11.13)
- **Fallback**: `C:\Users\ggarcia\AppData\Local\ESRI\conda\envs\arcgispro-py3-vscode\python.exe`
- Always quote Windows paths and include `.exe` extension
- Use forward slashes or escape backslashes in paths

## FFmpeg

- **Path**: `C:\Users\ggarcia\AppData\Local\Microsoft\WinGet\Packages\Gyan.FFmpeg_Microsoft.Winget.Source_8wekyb3d8bbwe\ffmpeg-8.0.1-full_build\bin\ffmpeg.exe`
- **Version**: 8.0.1
- Installed via `winget install ffmpeg`

## Setup

1. Add FAL_KEY to `.env` file (get from https://fal.ai/dashboard/keys)
2. Activate venv: `.venv\Scripts\activate`
3. Dependencies already installed: fal-client, requests, python-dotenv

## fal.ai API Models

| Service | Model ID | Purpose | Cost |
|---------|----------|---------|------|
| Nano Banana Pro | `fal-ai/nano-banana-pro` | Text-only slides | $0.15/image |
| Nano Banana Pro Edit | `fal-ai/nano-banana-pro/edit` | Slides with reference images | $0.15/image |
| Eleven Labs v3 | `fal-ai/elevenlabs/tts/eleven-v3` | Voiceover narration | ~$0.30/1000 chars |
| Kling 2.6 Pro | `fal-ai/kling-video/v2.6/pro/image-to-video` | Animated transitions | $0.35/video |

**5-slide presentation cost**: ~$2.65-3.00

## Available Reference Images

Located in `Reference Images/` folder:
- `Abonmarche Primary Logo Full Color Transparent BG.png`
- `Abonmarche A Transparent BG.png`
- `COP Logo.png`

## Voice Options

George (default), Aria, Rachel, Sam, Charlie, Emily

## Transition Styles

cinematic (default), zoom_blur, swipe, shatter, morph, particles, flip, wave, slide_left, fade_blur

## Development Guidelines

- Avoid emojis and special Unicode characters in scripts (Windows encoding issues)
- Store secrets in `.env` or config files, never in code
- `.env` is in `.gitignore`

## Project Structure

```
SlideShow-Agent/
  .env                    # FAL_KEY (not in git)
  requirements.txt        # Python dependencies
  Reference Images/       # Brand logos for slide generation
  output/                 # Generated videos (not in git)
  .claude/skills/slideshow-video-agent/
    SKILL.md              # Main skill definition
    scripts/              # Python modules
      orchestrator.py     # Main workflow coordinator
      slide_generator.py  # Nano Banana Pro integration
      voiceover.py        # Eleven Labs TTS integration
      transition_generator.py  # Kling video transitions
      video_assembler.py  # FFmpeg video assembly
      utils.py            # Shared utilities
    references/           # Documentation
      voice_options.md
      transition_styles.md
      workflow_guide.md
      api_reference.md
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GarrickGarcia)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/GarrickGarcia)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
