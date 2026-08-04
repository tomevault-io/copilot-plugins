## brasil-torrent

> Brasil Torrent is a wxPython + libtorrent 2.0.x BitTorrent client focused on accessibility for blind/visually impaired users (NVDA/JAWS/Narrator).

# Agent Configuration for Brasil Torrent

## Project Overview

Brasil Torrent is a wxPython + libtorrent 2.0.x BitTorrent client focused on accessibility for blind/visually impaired users (NVDA/JAWS/Narrator).

## Key Technologies

- **Python 3.12+** — Target runtime
- **wxPython 4.x** — Desktop GUI framework
- **libtorrent 2.0.x** — BitTorrent engine via `python-libtorrent`
- **accessible_output2** — Screen reader output (NVDA, JAWS, SAPI5)
- **pytest** — Testing framework

## Project Structure

```
brasil_torrent/
├── client/
│   ├── main.py              # App entry point, main frame
│   ├── config.py            # Settings manager (JSON-based)
│   ├── version.py           # Version info
│   ├── app_paths.py         # Resource path resolution
│   ├── core/
│   │   ├── torrent_engine.py  # libtorrent wrapper
│   │   ├── i18n.py           # Internationalization
│   │   └── utils.py          # Formatters (size, speed, eta, etc.)
│   ├── ui/
│   │   ├── accessible.py     # Accessible wx wrappers
│   │   ├── transfers.py      # Main torrent list panel
│   │   └── dialogs/
│   │       ├── add_torrent.py
│   │       ├── torrent_detail.py
│   │       ├── settings_dialog.py
│   │       ├── about_dialog.py
│   │       └── terms_dialog.py
│   ├── languages/
│   │   ├── pt-BR.json
│   │   └── en-US.json
│   └── tests/
│       ├── test_config.py
│       ├── test_i18n.py
│       ├── test_torrent_entry.py
│       ├── test_utils.py
│       └── test_app_paths.py
└── requirements.txt
```

## Architecture Patterns

- **I18n First** — All user-facing strings use `i18n.t("key")` pattern
- **Accessibility First** — Every control has `SetLabel()`, keyboard shortcuts, and screen reader output
- **Output Method** — `main_window.output(text)` speaks via accessible_output2 or logs if no screen reader
- **Settings** — Dict-based, saved to `data/config.json` via `settings["section"]["key"]`
- **Engine** — TorrentEntry dataclass wraps libtorrent handle; callbacks for add/remove/notification

## Testing

```bash
pytest client/tests/ -v
```

105 tests covering config, i18n, torrent entry state, utility functions, and app paths.

## Key Patterns

- All timers use 2-3 second intervals for UI updates
- Modal dialogs for settings, details, add torrent
- EVT_CHAR_HOOK for Alt+F4 interception (not EVT_CLOSE)
- Global hotkey Ctrl+Shift+B restores from tray via RegisterHotKey

---
> Source: [azurejoga/brasil-torrent](https://github.com/azurejoga/brasil-torrent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
