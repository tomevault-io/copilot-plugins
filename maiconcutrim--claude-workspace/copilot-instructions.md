## claude-workspace

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the app

```bash
python pomodoro/pomodoro.py
```

No dependencies beyond the Python standard library.

## Architecture

Single-file TUI application (`pomodoro/pomodoro.py`). Key sections:

- **Constants** (top of file): `WORK_MIN`, `SHORT_BREAK_MIN`, `LONG_BREAK_MIN`, `SESSIONS_BEFORE_LONG` — all timer durations are configured here.
- **`run_timer()`**: Core loop. Redraws the screen every 0.5 s using ANSI escape codes. Returns `"done"`, `"skip"`, or `"quit"`. Handles pause state internally via `pause_total` accumulator.
- **Cross-platform input**: `kbhit_nonblock()` / `read_key()` branch on `WINDOWS` (detected at import via `msvcrt`). On POSIX, stdin is set to `cbreak` mode for the duration of each timer.
- **Persistence**: `history.json` lives next to the script. `load_history()` / `save_history()` / `add_session()` are the only I/O functions. Only completed `"done"` work sessions increment `total_work_minutes`.
- **`main()`**: Orchestrates the work→break cycle. `session_count % SESSIONS_BEFORE_LONG == 0` triggers the long break.

## Customization

Edit the four constants at the top of `pomodoro/pomodoro.py` to change timer durations.

---
> Source: [maiconcutrim/claude-workspace](https://github.com/maiconcutrim/claude-workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
