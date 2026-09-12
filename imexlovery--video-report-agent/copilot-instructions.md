## video-report-agent

> This is an independent, lean product derived from video-evidence-agent.

# Video Report Agent

This is an independent, lean product derived from video-evidence-agent.
The Owner authorized replacing the historical V0/V1.1/V1.2 architecture on 2026-09-08.
Those historical contracts and evidence live in the original repository.

Keep the existing frontend, Bilibili ingestion, FFmpeg, MLX Whisper and Transcript Foundation.
Default to asr-only; retain optional subtitles/OCR/Fusion and Canonical Transcript.
Pi consumes the same canonical transcript projection regardless of input mode.
The generation path is Python → Pi RPC → video-report skill → DeepSeek → report.html.
Pi owns the agent loop, standard tools, session runtime and compaction.
Do not add planners, critics, revision workflows, multi-agent orchestration or compatibility layers.
Each generation runs in runs/<id>; never use project source as Pi's writable cwd.
This is a local tool with a workspace convention, not an OS sandbox.

Use the smallest sufficient change. Do not refactor unrelated code or add speculative infrastructure.
Use project-local .venv and uv run; update dependencies with uv add/remove and preserve uv.lock.
Run relevant tests. Never represent mocked checks as real end-to-end evidence.
Do not commit, push or deploy without an explicit request.

---
> Source: [imexlovery/video-report-agent](https://github.com/imexlovery/video-report-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-12 -->
