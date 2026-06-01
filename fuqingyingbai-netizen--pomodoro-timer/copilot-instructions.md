## pomodoro-timer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Run

```bash
D:\conda\python.exe 番茄钟.py
```

Or double-click `启动番茄钟.bat`. Requires Python 3 with tkinter (built into standard library).

## Architecture

Single-file Python tkinter desktop Pomodoro timer. No external dependencies.

**State model**: The `PomodoroApp` class holds all mutable state as instance attributes:
- `mode` — one of `'work'`, `'shortBreak'`, `'longBreak'`
- `time_left` / `total_time` — current countdown in seconds
- `completed` — total completed pomodoros (used modulo `longInterval` to determine long vs short break)
- `running` — timer active flag
- `settings` — dict loaded from `.pomodoro_settings.json` (auto-saved on settings dialog save)

**Timer mechanism**: Uses `tk.after()` for scheduling (`self.after_id`). No threading. The `tick()` method schedules itself every 1 second while `self.running` is True. `stop_timer()` cancels the pending `after` callback.

**UI refresh strategy**: Three tiers:
1. `draw_progress()` — redraws only the Canvas arc (called every tick, lightweight)
2. `refresh_mode()` — updates colors, tabs, and label when mode switches
3. `refresh_all()` — full refresh: mode + time text + progress + session dots + button text

**Arc rendering**: Uses `Canvas.create_arc(start=90, extent=-pct*360, style='arc')` with tags `'progress'`. Each tick deletes the `'progress'` tag group and redraws — avoids a persistent arc object that would need property mutation.

**Session dots**: Stored as per-dot mini `Canvas` widgets in `self.dot_canvases`. Dynamically rebuilt via `_build_dots()` when `longInterval` changes in settings.

**Settings persistence**: JSON file at `.pomodoro_settings.json` (gitignored). Defaults in `DEFAULTS` dict are merged with saved values on load.

**Keyboard shortcuts**: Space (toggle), R (reset), Right (skip), 1/2/3 (switch mode). Bound on root window via `<Key-N>` events.

## Git

Remote: `https://github.com/fuqingyingbai-netizen/pomodoro-timer`

```bash
git add . && git commit -m "..." && git push
```

---
> Source: [fuqingyingbai-netizen/pomodoro-timer](https://github.com/fuqingyingbai-netizen/pomodoro-timer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
