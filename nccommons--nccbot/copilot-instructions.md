## nccbot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

nccbot is a Python-based Wikimedia bot for managing and importing medical imaging content (primarily from Radiopaedia) to NC Commons (nccommons.org). It processes radiology studies, case images, and creates structured image sets on a MediaWiki platform.

## Commands

### Running Scripts
All scripts are executed via the Pywikibot wrapper:
```bash
python3 core8/pwb.py <module_path> [args] [study_ids...]

# Examples:
python3 core8/pwb.py fix_sets/new ask 109711     # Fix image set for study 109711 (interactive)
python3 core8/pwb.py mass/radio/get_studies test # Fetch study data from Radiopaedia
python3 core8/pwb.py infos/start debug           # Extract page info in debug mode
```

### Common Arguments
- `ask` - Interactive confirmation before making changes
- `debug` - Debug mode (limits processing count)
- `nodiff` - Skip diff display
- `nodb` - Skip database operations
- `noapi` - Skip API calls
- `test` - Test mode

### Job Framework (Wikimedia Toolforge)
```bash
tfj run --mem 1Gi --image python3.9 --command "$HOME/local/bin/python3 core8/pwb.py <script>"
```

### Shell Scripts
Sequential workflows are in shell scripts:
- `fix_sets/s.sh` - Sequential fix workflow
- `mass/eyerounds/eyerounds.sh`, `mass/usaid/usaid.sh` - Mass import workflows

## Architecture

### Core Directory: `ncc_core/`

**`api_bots/`** - Database and API abstraction layer:
- `db_bot.py` - SQLite wrapper using sqlite-utils (`LiteDB` class)
- `wiki_page.py` - Wikipedia/MediaWiki API loader with lru_cache
- `page_ncc.py` - NC Commons-specific API wrapper
- `user_account_new.py` - Credential management from INI files at `../ncc/confs/user.ini`

**`nccommons/`** - NC Commons functionality:
- `api.py` - Page creation, upload, query functions
- `api_upload.py` - File upload by URL/file
- `com.py` - Batch page creation from file lists

### Main Workflows

**`fix_sets/`** - Fix and organize image sets:
- `new.py` - Main workflow entry point for processing study IDs
- `bots/` - Core processing (stacks, study files, image info)
- `bots2/` - Advanced processing (set text generation, file movement, categories)

**`mass/`** - Mass import workflows:
- `radio/` - Radiopaedia.org imports (`get_studies.py` fetches study data)
- `eyerounds/`, `usaid/`, `atlas/`, `usask/` - Other medical image sources

**`sets_dbs/`** - Database operations:
- `main_db.py` - `DbClass` base for MySQL operations on Wikimedia ToolsDB

**`infos/`** - Page info extraction from NCC pages

### Key Patterns

**API Usage:**
```python
from api_bots.wiki_page import load_main_api
main_api = load_main_api("en", "wikipedia")
page = main_api.MainPage(title)
```

**SQLite Database:**
```python
from api_bots.db_bot import LiteDB
db = LiteDB(db_path)
db.create_table(table_name, fields, pk="id")
db.insert(table_name, data)
```

**MySQL Database (ToolsDB):**
```python
from sets_dbs.main_db import DbClass
class MyDb(DbClass):
    def __init__(self):
        self.table_name = 'table_name'
        self.create_table_query = 'CREATE TABLE IF NOT EXISTS ...'
        super().__init__(self.table_name, self.create_table_query)
```

## Configuration

- Credentials stored in INI files outside the repo at `../ncc/confs/`
- `.gitignore` excludes: `*.ini`, `*.json`, `*.sqlite`, `*.log`
- GitHub Actions auto-deploys to server on push to main via `.github/workflows/deploy.yml`

## Data Flow

1. `mass/radio/get_studies.py` fetches study data from Radiopaedia, caches to JSON files
2. `fix_sets/new.py` processes study IDs, generates wikitext for image sets
3. NC Commons API uploads/creates pages
4. SQLite tracks processed items locally; MySQL ToolsDB for persistent storage

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NCCommons) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
