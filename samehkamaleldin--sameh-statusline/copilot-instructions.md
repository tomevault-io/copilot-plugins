## sameh-statusline

> A single-file Python script (`statusline.py`) that renders a powerline-style status bar for Claude Code. Zero external dependencies — stdlib only, Python 3.10+.

# sameh-statusline — Development Guide

## Project overview

A single-file Python script (`statusline.py`) that renders a powerline-style status bar for Claude Code. Zero external dependencies — stdlib only, Python 3.10+.

## Architecture

- **Single file**: All logic lives in `statusline.py` — no packages, no modules
- **Data flow**: stdin JSON (from Claude Code) -> detection (git/stacks/tools) -> segment groups -> ANSI render -> stdout
- **Caching**: File-based cache at `~/.cache/sameh-statusline/` with TTL (30s general, 300s PR)
- **Progressive truncation**: Segments drop gracefully as terminal narrows

## Key conventions

- All subprocess calls go through `_run()` with timeouts — never hang
- `_sanitize()` all external input before rendering
- `_display_width()` for correct terminal width accounting (wide chars, zero-width)
- Colors use 256-color palette indices, not truecolor
- Nerd Font v3 glyphs via the `Icon` class

## Testing

Run manually with sample JSON:
```bash
echo '{"workspace":{"project_dir":"/tmp/test"},"context_window":{"context_window_size":200000,"used_percentage":25,"remaining_percentage":75},"cost":{"total_cost_usd":1.5,"total_duration_ms":120000},"model":{"display_name":"Opus 4.6"}}' | python3 statusline.py
```

## Packaging

- `pyproject.toml` defines the pip package with entry point `sameh-statusline`
- Build: `python3 -m build`
- The script must remain a single importable module (`statusline.py` at root)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/samehkamaleldin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/samehkamaleldin)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
