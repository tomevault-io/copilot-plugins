## claude-video

> Claude Video is an AI-powered video production suite for Claude Code. It provides 15 sub-skills covering the full video production pipeline from editing to AI generation.

# Claude Video: Project Instructions

## Overview

Claude Video is an AI-powered video production suite for Claude Code. It provides 15 sub-skills covering the full video production pipeline from editing to AI generation.

## Architecture

The skill follows the orchestrator pattern:
- `skills/video/SKILL.md` routes commands to sub-skills
- Each sub-skill handles one workflow (edit, transcode, caption, etc.)
- Scripts in `scripts/` provide execution capabilities (FFmpeg, Python)
- References in `references/` provide on-demand knowledge
- Agents in `agents/` handle complex multi-step tasks

## Key Conventions

- All FFmpeg commands use `-n` (no overwrite) by default
- Scripts output JSON for structured parsing
- Reference files are loaded on-demand, never all at startup
- SKILL.md files stay under 500 lines
- No credentials or API keys in the repository
- Python scripts use stdlib where possible

## Promo Pipeline

The promo pipeline at `promo-pipeline/` is a Remotion project that:
1. Searches Pixabay for stock footage
2. Downloads and preprocesses to 1080p 30fps
3. Analyzes frame contrast for adaptive text placement
4. Generates TTS voiceover via Gemini
5. Renders with transitions, music ducking, and sound effects

## Development

- Python scripts: argparse CLI, JSON output, error handling
- Shell scripts: `set -euo pipefail`
- TypeScript: Remotion patterns (useCurrentFrame, interpolate, spring)
- No em dashes in documentation, use colons or commas instead

---
> Source: [AgriciDaniel/claude-video](https://github.com/AgriciDaniel/claude-video) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
