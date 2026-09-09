## whiteboard

> This is the canonical entry point for Codex, Claude Code, OpenClaw, and other coding agents.

# Agent Guide

This is the canonical entry point for Codex, Claude Code, OpenClaw, and other coding agents.

## Objective

Install the project from a fresh clone, request only the required secrets, run the offline checks, generate the 30-second demo, and report verifiable output paths and metrics.

## Guardrails

- Never print, commit, or echo complete API keys.
- Never stage `auto-whiteboard/config/config.ini`, `skills/whiteboard-video-workflow/.env`, `output/`, or user transcripts unless explicitly requested.
- Preserve the locked production defaults in `README.md`.
- Reuse an existing `--project-dir` when resuming; do not regenerate expensive stages without a reason.
- Treat `scripts/` as the canonical public interface. Do not use historical commands from old commits or generated reports.

## Required Inputs

For video generation, ask for:

1. A MiniMax or 302.AI-compatible TTS key.
2. One supported image-provider key.

For cover generation, also ask for a 302.AI key stored as `AI302_KEY`. Claude is optional and is not required by the default workflow.

Supported image providers: `apimart_image2`, `kie_image2`, `t8_image2`, `macode_image2`.

## Install Sequence

Windows:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\bootstrap.ps1
.\.venv\Scripts\python.exe scripts\configure_keys.py
powershell -ExecutionPolicy Bypass -File .\scripts\doctor.ps1
powershell -ExecutionPolicy Bypass -File .\scripts\run_demo.ps1
```

macOS/Linux:

```bash
bash scripts/bootstrap.sh
./.venv/bin/python scripts/configure_keys.py
bash scripts/doctor.sh
bash scripts/run_demo.sh
```

If covers are requested, run `scripts/doctor.py --require-cover` before generation.

## Skills

- `skills/auto-whiteboard-video`: generate a complete narrated whiteboard video from a transcript.
- `skills/youtube-cover-generator`: generate YouTube and platform covers in the locked visual style.
- `skills/whiteboard-animation`: internal renderer used by the main pipeline.
- `skills/whiteboard-video-workflow`: internal storyboard and image-generation workflow.

Read the relevant `SKILL.md` before executing that workflow.

## Acceptance Criteria

The demo must create:

- `output/demo/latest/final_video.mp4`
- `output/demo/latest/composition_report.json`

Read the report and confirm:

- `output_width = 1920`
- `output_height = 1080`
- `single_line_subtitles = true`
- `output_av_delta_seconds < 0.1`

Before committing, run the checks listed in `AGENT_RUNBOOK.md` and leave unrelated user files untouched.

---
> Source: [shaomingchan/whiteboard](https://github.com/shaomingchan/whiteboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
