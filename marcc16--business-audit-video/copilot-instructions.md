## business-audit-video

> Generates a delivery package in `output/`:

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this plugin.

## Overview

Business Audit Video is a Claude Code plugin for generating professional business audit/consulting videos with AI-powered avatars. It combines web research, strategic script generation, AI voiceover (ElevenLabs), and talking head video (HeyGen) to create personalized pitch videos for companies.

**Input:** Company URL + optional job posting URL
**Output:** 60-second professional video with strategic analysis and avatar presenter

## Prerequisites

- **Python 3.10+** with `uv` package manager
- **Environment variables:**
  - `HEYGEN_API_KEY` - HeyGen API key for avatar video generation
  - `ELEVEN_LABS_API_KEY` - ElevenLabs API key for voiceover
  - `OPENAI_API_KEY` - OpenAI API key for script generation (GPT-4o)
  - `CLOUDINARY_URL` (optional) - For audio hosting, or use alternative

## Architecture

```
business-audit-video/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest
├── skills/
│   └── generate-audit-video/
│       ├── SKILL.md             # Main workflow and instructions
│       ├── scripts/
│       │   ├── research.py      # Company research orchestration
│       │   ├── script_generator.py  # AI-powered script generation
│       │   ├── generate_voiceover.py  # ElevenLabs TTS
│       │   ├── heygen_lipsync.py  # HeyGen avatar video generation
│       │   ├── extract_colors.py  # Corporate color extraction
│       │   ├── upload_audio.py   # Audio hosting utilities
│       │   └── orchestrator.py   # Main pipeline
│       └── prompts/
│           ├── research_prompt.txt  # Research analysis prompt
│           └── script_template.txt  # Script generation template
└── CLAUDE.md                    # This file
```

## Workflow

1. **Research Phase**: Analyzes company website and job posting (if provided) using Claude Code's WebSearch and WebFetch capabilities
2. **Script Generation**: Uses Claude API to generate strategic 60-second script based on research
3. **Color Extraction**: Extracts corporate colors from company website for branding
4. **Voiceover**: Generates professional audio with ElevenLabs (Spanish support)
5. **Avatar Video**: Creates talking head video with HeyGen using circular avatar style
6. **Final Composition**: Combines all elements into deliverable video package

## Usage

Invoke the skill with:

```bash
/business-audit-video:generate-audit-video --company-url "https://www.rsm.global/spain/es" --job-url "https://linkedin.com/jobs/view/123"
```

The orchestrator handles the entire pipeline automatically, with graceful error handling if any component fails.

## Output

Generates a delivery package in `output/`:
- `video.mp4` - Final video with avatar overlay
- `thumbnail.jpg` - Video thumbnail
- `script.json` - Complete script with timings
- `research.json` - Research data for reference
- `README.md` - Usage instructions

## Technical Notes

- **HeyGen avatar**: Uses pre-configured avatar ID `c3a34ee5ac854520b354702ad8bed061`
- **Avatar style**: Circular overlay positioned bottom-right
- **Audio hosting**: Audio must be publicly accessible for HeyGen API
- **Script structure**: Hook (0-10s) → Pain Points (10-25s) → Solution (25-40s) → Proof (40-55s) → CTA (55-60s)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marcc16) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
