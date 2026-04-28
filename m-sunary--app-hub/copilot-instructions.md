## app-hub

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

AppHub is a PyQt6 desktop dashboard that scans `~/Documents/workspace/python-workspace` for Python projects and lets you launch, stop, and manage them from a single UI. It runs as a standalone app — not a library.

## Commands

```bash
# Install dependencies
pip install -r requirements.txt   # PyQt6>=6.6.0

# Run the app
python main.py
```

No tests or build system are configured.

## Architecture

```
main.py                  # Entry point — creates QApplication and MainWindow
src/
  core/
    scanner.py           # AppScanner + AppInfo dataclass — all scan/launch/stop logic
  ui/
    main_window.py       # MainWindow — sidebar nav, category filter, search, grid layout
    app_card.py          # AppCard — individual project card widget (165px tall, 3-col grid)
    quick_bar.py         # QuickLaunchBar — top bar showing favorites + recent apps (max 8)
    styles.py            # Global QSS stylesheet (STYLESHEET constant)
```

### Key Design Decisions

**AppScanner** (`src/core/scanner.py`) is the core engine:
- Scans the hardcoded workspace path for subdirectories containing `main.py`, `requirements.txt`, or `README.md`
- App metadata (name, icon, color, category, description) is defined in the `APP_META` dict in `scanner.py` — keyed by directory name. Unknown projects get defaults.
- Tech stack is auto-detected by scanning `requirements.txt` against `TECH_KEYWORDS`.
- Favorites, hidden apps, and per-app category overrides persist in `~/.../python-workspace/.apphub.json`.
- Running status is detected via `pgrep -f "python.*<dir>/main.py"`. Apps are launched with `subprocess.Popen(['python3', 'main.py'])` in a new session.

**MainWindow** (`src/ui/main_window.py`):
- Workspace path is hardcoded: `WORKSPACE = os.path.expanduser('~/Documents/workspace/python-workspace')`
- A `QTimer` fires every 5 seconds to refresh running status without rescanning the filesystem.
- Filtering is done in `_get_filtered_apps()` combining nav state (`all`/`fav`/`recent`/`running`), category, and search text — all in-memory, no re-scan.

**AppCard** signals (all emit `AppInfo` objects):
`launch_requested`, `stop_requested`, `folder_requested`, `terminal_requested`, `favorite_toggled`

### Adding a New Project to the Dashboard

Add an entry to `APP_META` in `src/core/scanner.py`:
```python
'your-dir-name': {
    'name': 'Display Name',
    'icon': '🔧',
    'color': '#hexcolor',
    'category': '工具',   # 金融 | 学习 | 工具 | Web
    'desc': 'Short description',
},
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/M-sunary) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
