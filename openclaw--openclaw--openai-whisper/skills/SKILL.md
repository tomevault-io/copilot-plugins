---
name: openai-whisper
description: Local speech-to-text with the Whisper CLI (no API key). Use when this capability is needed.
metadata:
  author: openclaw
---

# Whisper (CLI)

Use `whisper` to transcribe audio locally.

Quick start

- `whisper /path/audio.mp3 --model medium --output_format txt --output_dir .`
- `whisper /path/audio.m4a --task translate --output_format srt`

Notes

- Models download to `~/.cache/whisper` on first run.
- `--model` defaults to `turbo` on this install.
- Use smaller models for speed, larger for accuracy.

---
> Converted and distributed by [TomeVault](https://tomevault.io) | [Claim this content](https://tomevault.io/claim/openclaw/openclaw)
<!-- tomevault:3.0:skill_md:2026-04-07 -->
