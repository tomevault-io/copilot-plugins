## yt-agent

> Terminal-first YouTube CLI built on yt-dlp. Search, download, catalog (SQLite+FTS5), clip extraction, Textual TUI.

# yt-agent — Agent Guide

## What this is
Terminal-first YouTube CLI built on yt-dlp. Search, download, catalog (SQLite+FTS5), clip extraction, Textual TUI.

## Dev commands
- Install: `uv sync --dev`
- Run tests: `uv run pytest --cov` (fallback: `python -m pytest --cov`)
- Lint: `uv run ruff check .` (fallback: `python -m ruff check .`)
- Type check: `uv run mypy`
- Build: `uv build`

## Key pattern and convention guide

### SQL safety
- Always use `?` placeholders for SQL parameters
- LIKE queries use `ESCAPE '\'`
- FTS5 queries are sanitized per-token via `_fts_query()` in `catalog.py`

### Security pattern
- All user-facing text goes through `sanitize_terminal_text()` from `security.py`
- File permissions: 0o700 dirs, 0o600 files on POSIX
- URL allowlisting via `ALLOWED_YOUTUBE_HOSTS` in `yt_dlp.py`
- Subprocess calls are list-based only, never `shell=True`

### CLI convention
- Every command wraps logic in `_run_guarded()` which catches `YtAgentError`, `sqlite3.Error`, `KeyboardInterrupt`
- `dict[str, Any]` for JSON payloads in cli.py is intentional — do not replace with TypedDict
- Exit codes: OK=0, DEPENDENCY=3, INPUT=4, CONFIG=5, EXTERNAL=6, BUSY=7, STORAGE=8, INTERRUPTED=130
- `operation_lock()` prevents concurrent mutations — acquire for writes, not reads

### Output convention
- Read commands support `--output table|json|plain`
- Mutation commands support `--output json`, `--dry-run`, `--quiet`
- JSON error envelopes: `{schema_version, status, exit_code, error_type, message}`

### Model convention
- All models are `@dataclass(frozen=True)` — immutable
- `VideoInfo.from_yt_dlp()` handles coercion from raw yt-dlp JSON

## Areas to watch with extra care

### High risk areas
- `cli.py` (~2350 lines) is the densest module — be careful with imports and the `_run_guarded` pattern
- `catalog.py` schema changes must preserve backward compatibility
- Tests heavily use `monkeypatch.setattr("yt_agent.cli.<name>", ...)` — any symbol moved from cli.py must be re-exported so the monkeypatch path still resolves
- `_fts_query()` sanitization is security-critical — test adversarial inputs
- `operation_lock` is flock-based (POSIX) / msvcrt (Windows) — platform-specific behavior

### Build verification
After any changes, verify the full chain:
```bash
uv run ruff check .
uv run mypy
uv run pytest --cov
uv build
```

---
> Source: [jvogan/yt-agent](https://github.com/jvogan/yt-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
