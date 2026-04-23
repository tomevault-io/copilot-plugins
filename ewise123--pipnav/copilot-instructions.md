## pipnav

> Fallout Pip-Boy themed TUI for navigating and launching dev projects from WSL. Built with Python + Textual.

# CLAUDE.md — PipNav

Fallout Pip-Boy themed TUI for navigating and launching dev projects from WSL. Built with Python + Textual.

## Run

```bash
cd ~/projects/PIPNAV
source .venv/bin/activate
pipnav
```

## Test

```bash
pytest
pytest -s  # with output
```

## Structure

- `pipnav/core/` — business logic, no Textual imports, independently testable
- `pipnav/ui/` — one widget per file, all CSS in `app.tcss`
- `tests/` — unit tests for core modules

## Conventions

- Python 3.10+, type hints, `pathlib.Path` for filesystem
- Frozen dataclasses for all data types (immutability)
- `@work(thread=True)` for blocking I/O (git, filesystem)
- 4 color schemes via Textual themes: green, amber, blue, white (press `p` to cycle)
- Config/state in `~/.pipnav/` as JSON
- Errors: never crash, degrade gracefully, log to `~/.pipnav/debug.log`

## Keybindings

- `j/k` navigate, `h/l` switch focus left/right
- `Enter` drill into folder, `Backspace` go back
- `v` VS Code, `c` Claude Code, `r` resume Claude
- `Tab` or `1-5` switch tabs (STAT/FILES/LOG/CONSOLE/INV)
- `/` search, `t` tag, `n` note, `p` color scheme, `~` sound effects, `?` help, `q` quit

## Future Ideas

- **Digits widget** — large 3x3 numeric displays for project count or session count in header
- **textual-canvas** — half-block pixel drawing for mini project maps or decorative borders
- **Typewriter text rendering** — character-by-character reveal when selecting projects in detail panel

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ewise123) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
