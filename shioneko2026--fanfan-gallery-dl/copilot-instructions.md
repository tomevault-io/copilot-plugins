## fanfan-gallery-dl

> A PyQt6 desktop GUI for downloading media from artist subscription platforms (Pixiv Fanbox, Patreon, Fantia, SubscribeStar) using gallery-dl as the backend.

# FanFan Gallery-DL - Claude Code Context

## Project Overview
A PyQt6 desktop GUI for downloading media from artist subscription platforms (Pixiv Fanbox, Patreon, Fantia, SubscribeStar) using gallery-dl as the backend.

## Tech Stack
- **Frontend**: PyQt6 (desktop GUI)
- **Backend**: gallery-dl (CLI tool in /bin/)
- **Database**: SQLite (data/appdata.db)
- **Language**: Python 3.9+
- **Platform**: Windows (credential storage via Windows Credential Manager)

## Key References
- Original inspiration: https://github.com/KJHJason/Cultured-Downloader
- Backend: https://github.com/mikf/gallery-dl

## Project Structure
- `main.py` - Entry point
- `ui/` - PyQt6 UI modules (main_window, dashboard, downloads, creators, settings)
- `core/` - Business logic (gallery_dl_manager, download_queue, credential_manager)
- `db/` - Database layer (SQLite operations)
- `bin/` - gallery-dl executables
- `config/` - Configuration files
- `data/` - Application data (appdata.db)

## Development Guidelines
1. Follow existing PyQt6 patterns in ui/ modules
2. Use database.py for all database operations
3. Credentials stored in Windows Credential Manager (never in files)
4. Gallery-dl runs as subprocess, never imported as Python module
5. Support Japanese text with pykakasi for romaji conversion
6. File naming patterns use tokens like {creator_name}, {creator_jp}, {date}, {category}

## Current Focus
- Implementing core/gallery_dl_manager.py
- Implementing core/download_queue.py  
- Connecting UI to backend functionality
- Testing credential manager with real platforms

## Testing
- Run: `python main.py` for GUI
- Run: `python test_gallery_dl.py` for backend tests
- Run: `test_run.bat` for quick launch

## Important Notes
- gallery-dl.exe is auto-downloaded on first run
- Database schema in db/database.py
- See *.md files for implementation progress and design decisions

---
> Source: [shioneko2026/fanfan-gallery-dl](https://github.com/shioneko2026/fanfan-gallery-dl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
