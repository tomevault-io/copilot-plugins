## ytune

> - **Package Manager:** Always use `pixi`. It manages both Python and system libraries like `libmpv`.

# GEMINI.md

## Core Mandates
- **Package Manager:** Always use `pixi`. It manages both Python and system libraries like `libmpv`.
- **Versioning:** Source of truth is `pyproject.toml`.
- **Authentication:** Required once via `pixi run ytmusicapi browser`. Result must be at `~/.config/ytune/auth.json`.

## Technical Stack
- **UI:** Textual (TUI)
- **Audio:** `yt-dlp` (extraction) + `mpv` (playback via bundled libmpv)
- **Data:** `ytmusicapi` + Pydantic models (`src/ytune/models.py`)
- **DB:** SQLite (`src/ytune/db.py`)

## Essential Commands
```bash
pixi run ytune         # Run App
pixi run test          # Run Tests
pixi add <package>     # Add Conda Dependency
pixi add --pypi <pkg>  # Add PyPI Dependency
```

## Architectural Pattern
- **Async TUI:** Background tasks must use `@work(thread=True)` and `self.call_from_thread()` for UI updates.
- **Player:** Wrapped in `src/ytune/player.py`, uses `libmpv` (bundled via pixi).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NestumMilo-isFezan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NestumMilo-isFezan)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
