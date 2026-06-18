## jre-vidget

> Teaching document for AI coding agents. Describes capabilities, workflows, guardrails,

# SKILL.md — jre-vidget

Teaching document for AI coding agents. Describes capabilities, workflows, guardrails,
and trigger conditions for working with the `vidget` CLI tool.

---

## Capabilities

- Download a single video from a URL (Fox News, YouTube, Brightcove HLS, etc.)
- List available formats for a URL without downloading
- Batch-download multiple URLs from a text file
- Manage persistent configuration (output directory, default quality, retries)
- Pre-flight check for yt-dlp and ffmpeg availability

---

## CLI Grammar

```
vidget [GLOBAL OPTIONS] COMMAND [ARGS] [COMMAND OPTIONS]

Commands:
  download  Download a single video URL
  batch     Batch-download URLs from a text file
  formats   List available formats for a URL
  config    show | set | reset — manage ~/.vidget/config.json
  check     Verify yt-dlp and ffmpeg are available
```

Global options available on all commands:
- `--json`           Machine-readable JSON output on stdout
- `--quiet` / `-q`   Suppress progress indicators (stderr)
- `--version`        Print version and exit

---

## Workflows

### 1. Download a video (human mode)
```bash
vidget download "https://video.foxnews.com/..." --output ~/Downloads
```

### 2. Download a video (agent / machine mode)
```bash
# Always use --json when parsing output programmatically
result=$(vidget download "https://video.foxnews.com/..." --output ~/Downloads --json)
echo $result | jq .data.filepath
```

### 3. Download with specific quality
```bash
vidget download "https://..." --quality best --format mp4 --output ~/Downloads --json
```

### 4. Batch download from file
```bash
# urls.txt: one URL per line, # for comments
vidget batch urls.txt --output ~/Downloads --json
```

### 5. List formats before downloading
```bash
vidget formats "https://..." --json
# Returns: {"ok": true, "data": {"formats": [{"id": "...", "ext": "mp4", "quality": "1080p"}, ...]}}
```

### 6. Check dependencies
```bash
vidget check --json
# Returns: {"ok": true, "data": {"ytdlp": true, "ffmpeg": true}}
# Exit 1 if yt-dlp is missing; exit 0 with warning if only ffmpeg missing
```

### 7. Non-interactive / headless download
```bash
# --yes skips all confirmation prompts; required when not running in a TTY
vidget download "https://..." --output ~/Downloads --yes --no-overwrite --json
```

### 8. Configure default output directory
```bash
vidget config set output_dir ~/Videos
vidget config show --json
```

---

## Exit Codes

| Code | Meaning | Agent action |
|------|---------|-------------|
| `0` | Success | Continue |
| `1` | General failure | Log error, stop |
| `2` | Bad arguments | Fix command, retry |
| `3` | Auth / permission error | Check credentials, stop |
| `4` | Transient error (timeout, rate limit) | Retry with backoff |
| `5` | File exists + `--no-overwrite` set | Skip or remove `--no-overwrite` |
| `130` | Interrupted (Ctrl-C) | Stop |

---

## Guardrails

| Invariant | Description |
|-----------|-------------|
| No interactive prompts in headless mode | If stdin is not a TTY, no prompts are shown |
| `--json` produces parse-clean stdout | Only valid JSON on stdout; Rich goes to stderr |
| Never commit downloaded media | `*.mp4`, `*.mkv`, etc. are in `.gitignore` |
| No real network in tests | All yt-dlp calls are mocked in unit and integration tests |
| Engine never imports ui.py | `engine.py` is importable without Rich installed |
| Config path is overridable | Tests use `tmp_path`, never touch `~/.vidget/config.json` |

---

## Trigger Conditions

Use `vidget` when an agent needs to:
- Retrieve video or audio from a URL that isn't accessible via a direct file download
- List video formats before deciding which quality to download
- Process a batch of URLs in a single command without N+1 shell invocations
- Verify the local environment has required media tools installed

Do **not** use `vidget` for:
- Text/image content scraping (not a web scraper)
- Live stream recording (not supported)
- Anything that requires a logged-in browser session without saved cookies

---

## JSON Output Shape

All `--json` responses follow this envelope:

```json
// Success
{
  "ok": true,
  "schemaVersion": 1,
  "type": "command.result",
  "data": { ... }
}

// Failure
{
  "ok": false,
  "schemaVersion": 1,
  "type": "command.result",
  "error": {
    "code": "download_failed",
    "message": "Human-readable description",
    "retryable": false
  }
}
```

The `retryable` field in error objects maps directly to exit code semantics:
`true` → exit code `4` (safe to retry); `false` → exit code `1` or `3`.

---

## Development Quick Reference

```bash
uv sync --extra dev          # Install all dependencies
uv run pytest                # Run full test suite
uv run pytest tests/unit -x  # Unit tests, stop on first failure
uv run ruff check src/       # Lint
uv run ruff format src/      # Format
uv run mypy src/ --strict    # Type check
make test                    # Equivalent to uv run pytest
make lint                    # ruff check + mypy
```

---
> Source: [jreakin/jre-vidget](https://github.com/jreakin/jre-vidget) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
