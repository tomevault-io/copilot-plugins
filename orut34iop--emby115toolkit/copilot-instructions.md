## emby115toolkit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Emby115Toolkit is a Python GUI utility for the 115 cloud disk + CloudDrive2 + Emby media server workflow. It creates symlinks, mirrors directory trees, manages Emby libraries, and merges scraped metadata with video files.

## Running the Application

Two entry points exist; choose based on the target platform:

- **Windows (tkinter):** `python main.py`
  - Uses `tkinterdnd2` for drag-and-drop.
  - Creating symlinks requires Administrator privileges on Windows.
- **macOS (PyQt5):** `python qt_main.py`
  - Uses native PyQt5 drag-and-drop.
  - No Administrator privileges needed for symlinks.

Both entry points share the same `config.yaml` and backend logic in `autosync/` and `emby/`.

## Installing Dependencies

```bash
pip install -r requirements.txt
# For macOS/PyQt5 version also install:
pip install PyQt5
```

## Building the Executable

```bash
build.bat
```

This runs `pyinstaller --clean build.spec` to produce `dist/Emby115Toolkit.exe`. The spec bundles `tkinterdnd2/tkdnd` data files and hidden imports required by the tkinter version.

## Testing

The project uses pytest with markers defined in `pytest.ini`:

```bash
pytest                              # Run all tests
pytest -m unit                     # Run only unit tests
pytest -m "not integration"        # Skip integration tests
pytest tests/unit/test_symlink_creator.py  # Run a single test file
pytest -k test_name                # Run tests matching a name pattern
```

Markers: `unit`, `integration`, `slow`, `gui`, `thread`, `emby`, `file_system`.

**Important for test authors:** `Config` is a singleton. `tests/conftest.py` provides an `isolate_config` autouse fixture that resets `Config._instance` and `Config._config` before/after every test. Use the `mock_config` fixture when you need a `Config` backed by a temporary directory.

## Project Architecture

### Dual GUI Frontends

The project maintains **two completely separate GUI layers** that do not import from each other:

- `tabs/` — tkinter tab implementations used by `main.py`. Each tab inherits from `BaseTab` which provides common widgets (path entries, log frames, drag-and-drop helpers).
- `qt_gui/` — PyQt5 tab implementations used by `qt_main.py`. Each tab is a `QWidget` subclass with native Qt drag-and-drop.

When adding a new feature or fixing a UI bug, **check both `tabs/` and `qt_gui/`** for the corresponding implementation. They are kept in sync manually.

### Tab-to-Backend Mapping

| Feature | tkinter Tab | PyQt5 Tab | Backend Module(s) |
|---|---|---|---|
| 导出软链接 | `tabs/export_symlink_tab.py` | `qt_gui/export_tab.py` | `autosync/SymlinkCreator.py`, `autosync/MetadataCopyer.py` |
| 文件夹操作 | `tabs/folder_operator_tab.py` | `qt_gui/folder_tab.py` | `autosync/SymlinkDeleter.py`, `emby/EmbyOperator.py` |
| Emby影剧查重 | `tabs/check_duplicate_tab.py` | `qt_gui/duplicate_tab.py` | `emby/EmbyOperator.py` |
| 文件合并 | `tabs/merge_files_tab.py` | `qt_gui/merge_tab.py` | `autosync/FileMerger.py` |
| Emby合并版本 | `tabs/merge_version_tab.py` | `qt_gui/version_tab.py` | `emby/EmbyOperator.py` |
| Emby更新流派 | `tabs/update_genres_tab.py` | `qt_gui/genres_tab.py` | `emby/EmbyOperator.py` |
| 115目录树镜像 | `tabs/mirror_115_tree_tab.py` | `qt_gui/mirror_tab.py` | `autosync/TreeMirror.py` |

### Backend Modules (`autosync/`)

Core business logic lives in `autosync/` and is shared by both GUIs:

- `SymlinkCreator.py` — Multi-threaded symlink/strm creation with optional path replacement.
- `MetadataCopyer.py` — Copies metadata files (nfo, posters, subtitles) alongside symlinks.
- `TreeMirror.py` — Parses a 115-exported directory tree text file and recreates an empty file tree locally.
- `FileMerger.py` — Moves video files into folders that contain matching nfo files.
- `SymlinkDeleter.py` — Folder cleanup utilities (delete symlinks, delete video files).

### Emby Integration (`emby/`)

- `EmbyOperator.py` — Single module wrapping Emby Server API calls. Handles duplicate checking (by TMDB ID), version merging, and genre translation (English → Chinese).

### Shared Utilities (`utils/`)

- `config.py` — Singleton `Config` class managing `config.yaml`. Uses a recursive merge strategy so new default keys are automatically added to existing user configs. Resolves `config_dir` to the EXE directory when `sys.frozen` is True, otherwise the project root.
- `logger.py` — Thread-safe `setup_logger()` that outputs to both a tkinter `Text` widget (via a queued batch handler) and rotating log files.
- `history_entry.py` — Helper for history-aware input widgets.
- `listdir.py` — Cross-platform file listing helper.

### Configuration (`config.yaml`)

Runtime configuration is stored in YAML at the project root (or next to the EXE when packaged). Sections correspond to tabs/features:

- `export_symlink` — source folders, target folder, suffixes, thread count, path replacement settings.
- `merge_file` — scrap folder and target folder.
- `merge_version`, `update_genres` — Emby URL and API key.
- `mirror_115_tree` — tree file path and export folder.
- `last_tab_index` — remembers the active tab across restarts.

**Do not commit `config.yaml`** — it is gitignored because it contains user-specific paths and API keys.

### Rust Component

A minimal `Cargo.toml` and `src/main.rs` exist but are effectively unused scaffolding (prints "Hello, world!"). The active codebase is entirely Python.

---
> Source: [orut34iop/Emby115Toolkit](https://github.com/orut34iop/Emby115Toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
