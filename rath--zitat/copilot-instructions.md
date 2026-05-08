## zitat

> YouTube clip subtitle pipeline. Single file `zitat.py`, stdlib only, no external packages.

# zitat

YouTube clip subtitle pipeline. Single file `zitat.py`, stdlib only, no external packages.

## Structure

```
zitat.py          # Entire pipeline (stdlib only)
```

## Pipeline (6 steps)

1. `yt-dlp` — Download video (max 1024px width, `--download-sections` + `--force-keyframes-at-cuts` when start/duration specified)
2. `ffmpeg` — Extract audio (16kHz mono WAV)
3. `whisper-cli` — Transcribe to SRT
4. `claude -p` — Translate subtitles (must filter out `CLAUDE_CODE_ENTRYPOINT` env var)
5. `$EDITOR` — Human review of translated subtitles (`--no-review` to skip)
6. `ffmpeg` — Burn subtitles (libass subtitles filter)

## Configuration

Whisper paths are managed via `.env` file (same directory as script). Priority: CLI options > shell env vars > `.env`.

## External tools

- `yt-dlp`, `ffmpeg`, `claude` — resolved from PATH
- `whisper-cli` — `WHISPER_BIN` (default: `whisper-cli` from PATH)
- Whisper model — `WHISPER_MODEL` (required, no default)

## Notes

- ffmpeg subtitles filter requires escaping `:` → `\:` in paths (libass parser)
- `extract_srt()` handles Claude responses wrapped in code fences or extra commentary
- Temp files go to `tempfile.mkdtemp(prefix="zitat_")`, cleaned up in `finally`
- `--keep-tmp` preserves intermediate files for debugging

---
> Source: [rath/zitat](https://github.com/rath/zitat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
