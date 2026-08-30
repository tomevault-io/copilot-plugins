## tinybookshelf

> Lightweight Qt5 Widgets audiobookshelf client for 32-bit low-end Linux (Acer Aspire One D250, Atom N270, 1GB RAM, Debian 12). Also builds on macOS for development.

# tinybookshelf

Lightweight Qt5 Widgets audiobookshelf client for 32-bit low-end Linux (Acer Aspire One D250, Atom N270, 1GB RAM, Debian 12). Also builds on macOS for development.

## Build

```bash
# Mac (development)
mkdir -p build && cd build
cmake .. -DCMAKE_PREFIX_PATH=$(brew --prefix qt@5)
make -j$(sysctl -n hw.ncpu)

# Debian 12 (target)
sudo apt install qtbase5-dev qtmultimedia5-dev libqt5svg5-dev cmake g++
mkdir -p build && cd build
cmake ..
make -j2
```

## Test

```bash
cd build && ctest --output-on-failure
```

7 test suites: test_dummy, test_configmanager, test_apiclient, test_audioengine, test_covercache, test_librarymodel, test_bookmarkmanager.

## Run

```bash
# Config file at ~/.config/tinybookshelf/config.ini (Linux)
# or ~/Library/Preferences/tinybookshelf/config.ini (macOS)
# See config.ini.example for format

./tinybookshelf              # uses default config path
./tinybookshelf /path/to/config.ini  # custom config path
```

## Architecture

Single-binary Qt5 Widgets app. No QML.

- **ConfigManager** — reads server URL + credentials from INI file
- **ApiClient** — REST client with JWT auth, JSON parsers for all audiobookshelf endpoints
- **AudioEngine** — wraps QMediaPlayer, multi-track audiobook support, chapter navigation
- **ProgressSync** — pushes playback position to server every 60s
- **CoverCache** — disk + QPixmapCache cover art caching
- **LibraryModel** — QAbstractListModel with local text filtering
- **MainWindow** — toolbar, stacked content (library grid / item detail), player bar
- **Theme** — dark charcoal (#1e1e2e) + amber (#e5a649) global stylesheet

## Key decisions

- **No Socket.IO** — polls REST API instead, simpler and sufficient for single-user
- **No server-side search** — search field does instant local filtering of loaded items
- **supportedMimeTypes** in playback request — server direct-plays common formats, HLS transcodes exotic ones
- **QToolBar quirk** — widget visibility must be toggled via QAction, not the widget itself
- **QComboBox on macOS** — needs custom QStyledItemDelegate for item sizing; stylesheets alone don't work
- **Icons** — Heroicons SVGs in resources/icons/, compiled via Qt resource system (.qrc)

## Project structure

```
src/
  main.cpp, ConfigManager, ApiClient, AudioEngine, ProgressSync,
  CoverCache, LibraryModel, MainWindow, LibraryView, ItemDetailView,
  PlayerBar, BookmarkDialog, BookmarkManager, Theme
tests/
  QTest suites for non-UI modules
resources/
  icons/ — Heroicons SVGs (recolored for dark theme)
  resources.qrc — Qt resource file
docs/superpowers/
  specs/ — design spec
  plans/ — implementation plan
```

---
> Source: [minorbug/tinybookshelf](https://github.com/minorbug/tinybookshelf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
