## lifeos

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**LifeOS** is a personal GTD (Getting Things Done) system with Todoist integration. The system enables natural language task creation, automated project management, and comprehensive life tracking.

---

## Quick Commands

```bash
# Todoist Task Management
./lifeos 'your task description'    # Natural language task creation
./lifeos setup                       # Setup Todoist API Token
./lifeos test                        # Test Todoist connection
./lifeos setup-goals                 # Quick setup for fitness/career/English goals
./lifeos fitness                     # Send workout plan
./lifeos export-tasks                # Export all tasks to JSON
./lifeos list-tasks                  # List all tasks

# Anki Sync (Notion → Anki)
./lifeos setup-anki [PAGE_ID]        # Create Anki Cards database in Notion
./lifeos sync-anki                   # Sync Notion to Anki (.apkg)
./lifeos sync-anki --dry-run         # Test sync without changes (safe mode)

# Eudic Sync (欧路词典 → Notion → Anki)
./lifeos sync-eudic                  # Sync Eudic vocabulary to Notion
./lifeos test-eudic                  # Test Eudic API connection (dry-run)
./lifeos sync-eudic --limit 10       # Test with limited number of words

# Life Tracking (Logseq Integration)
./lifeos today                       # Initialize today's journal
./lifeos log work 'content' 8 '2h'  # Log activity with rating and duration
./lifeos report                      # Generate weekly report
./lifeos sync                        # Git sync Logseq

# AI Advisor
./lifeos analyze                     # Analyze life patterns from historical data
./lifeos plan                        # Generate optimized plans based on analysis
```

---

## Core Components

### 1. Todoist Manager (`scripts/todoist_manager.py`)
Full-featured Todoist API integration:
- Create/manage tasks, projects, and labels
- Batch task operations
- Data export for analysis
- Pre-configured fitness/career/English templates

**Key Features:**
- Direct REST API integration (no email required)
- Cross-platform sync (Windows, Mac, iPhone, Android)
- Full programmatic control via Python
- Habit tracking and recurring tasks

### 2. Personal Assistant (`scripts/personal_assistant.py`)
Natural language task processing:
- Parse user input into structured tasks
- Auto-categorize by project and priority
- Estimate task duration
- Send to Todoist via API

### 3. Goals Setup (`scripts/setup_goals.py`)
Quick initialization for three major goals:
- **Fitness (work-out)**: 6 workout tasks
- **Career (job-hunt)**: 8 job search tasks
- **English (speak-up)**: 10 learning tasks

### 4. Logseq Tracker (`scripts/logseq_tracker.py`)
Daily journaling and life tracking:
- Template-based journal initialization
- Activity logging with ratings
- Weekly report generation
- Git synchronization

### 5. AI Advisor (`scripts/ai_advisor.py`)
Pattern analysis and recommendations:
- Analyze historical data from Logseq
- Generate optimization suggestions
- Create personalized plans

### 6. Anki Sync Manager (`scripts/sync_notion_anki.py`)
Notion to Anki synchronization system:
- Query unsynced cards from Notion "Anki Cards" database
- Generate .apkg files using genanki
- Send to Telegram via Bot API
- Update Notion sync status automatically
- Support for multiple decks and tags
- Prevent duplicate cards with stable GUIDs

**Key Features:**
- Uses Notion API 2025-09-03 (latest version)
- GitHub Actions automation (daily at 8am Beijing time)
- Dry-run mode for testing
- Cross-platform Anki import (mobile & desktop)

### 7. Eudic Sync Manager (`scripts/sync_eudic_notion.py`)
Eudic (欧路词典) vocabulary synchronization to Notion:
- Fetch vocabulary from Eudic API automatically
- Parse word, phonetics, and definitions
- Add to Notion "Anki Cards" database
- Track sync state to avoid duplicates
- Batch and incremental sync support

**Key Features:**
- Official Eudic OpenAPI integration
- Auto-tags: "欧路", "vocabulary"
- Deck: Vocabulary (customizable)
- Supports --limit flag for testing
- GitHub Actions automation (runs before Anki sync)
- Format: Front (word) + Back (phonetic + definition)

---

## Configuration Files

- `config/todoist_config.json` - Todoist API settings and project mappings
- `config/assistant_profile.json` - Assistant behavior configuration
- `config/logseq_templates.json` - Logseq journal templates
- `config/anki_sync_config.json` - Anki sync behavior and deck settings
- `config/eudic_config.json` - Eudic (欧路词典) API token and sync settings
- `notion-kit/.env` - Notion and Telegram credentials

### Todoist Project Mapping

```json
{
  "fitness": "work-out",    // Workout and fitness
  "career": "job-hunt",     // Job searching
  "english": "speak-up",    // English learning
  "errands": "Errands",     // Daily errands and life tasks (shopping, cleaning, etc.)
  "inbox": "Inbox"          // Default inbox for uncategorized tasks
}
```

### Labels

- `urgent` - Urgent tasks (red)
- `important` - Important tasks (orange)
- `routine` - Daily routines (grey)
- `habit` - Habit formation (green)

---

## Project Structure

```
LifeOS/
├── lifeos                        # Main entry point
├── CLAUDE.md                     # This file
├── README.md                     # Project readme
│
├── .github/
│   └── workflows/
│       └── sync-anki.yml         # GitHub Actions for daily sync
│
├── scripts/
│   ├── todoist_manager.py        # Todoist API core
│   ├── personal_assistant.py     # NLP task parser
│   ├── setup_goals.py            # Goal templates
│   ├── logseq_tracker.py         # Life tracking
│   ├── ai_advisor.py             # AI recommendations
│   ├── sync_notion_anki.py       # Anki sync script
│   └── setup_anki_database.py    # Anki database setup
│
├── config/
│   ├── todoist_config.json       # Todoist settings
│   ├── assistant_profile.json    # Assistant config
│   ├── logseq_templates.json     # Journal templates
│   └── anki_sync_config.json     # Anki sync settings
│
├── docs/                         # Documentation
│   ├── ANKI_SETTINGS_FINAL.md    # Optimized Anki settings guide
│   ├── ANKI_SYNC_QUICKSTART.md   # Anki sync quickstart
│   └── NOTION_ANKI_SYNC_PLAN.md  # Anki sync implementation plan
│
├── notion-kit/
│   ├── .env                      # Notion & Telegram credentials
│   └── notion_wrap.py            # Notion API wrapper (2025-09-03)
│
├── data/                         # Local data storage
│   ├── anki_sync_*.apkg          # Generated Anki packages
│   └── anki_sync_state.json      # Sync state tracking
│
├── archive/                      # Historical documents
│   ├── TODOIST_QUICKSTART.md     # Todoist setup guide (archived)
│   └── ANKI_SETTINGS_*.md        # Old Anki settings versions
│
└── knowledge/                    # Personal knowledge base
```

---

## Python Dependencies

```bash
pip install todoist-api-python    # Official Todoist SDK
pip install genanki               # Anki package generator
pip install notion-client         # Notion API SDK
pip install requests              # HTTP requests
pip install python-dotenv         # Environment variables
```

**Standard libraries used:**
- `json`, `os`, `sys`, `pathlib` - System operations
- `datetime`, `statistics` - Data processing
- `subprocess`, `argparse` - CLI operations
- `hashlib` - GUID generation

---

## Integration Points

### Todoist API
- **Authentication**: Bearer token in `config/todoist_config.json`
- **API Version**: REST API v2
- **Endpoint**: `https://api.todoist.com/rest/v2`
- **Features**: Projects, labels, priorities, due dates, recurring tasks

### Notion API
- **Authentication**: Integration token in `notion-kit/.env`
- **API Version**: 2025-09-03 (latest)
- **Endpoint**: `https://api.notion.com/v1`
- **Features**: Database queries, page updates, data_source_id support
- **Database**: "Anki Cards" with Front, Back, Deck, Tags, Source fields

### Telegram Bot API
- **Authentication**: Bot token + Chat ID in `notion-kit/.env`
- **Endpoint**: `https://api.telegram.org/bot{token}`
- **Features**: Send .apkg files as documents
- **Setup**: Create bot via @BotFather, get Chat ID via @userinfobot

### Logseq
- **Path**: `/root/Documents/logseq/`
- **Format**: Markdown-based daily journals
- **Sync**: Git version control

---

## Workflow Examples

### Creating Tasks

**Natural language:**
```bash
./lifeos '明天要准备面试资料和更新简历'
# Parses into 2 tasks, assigns to job-hunt project
```

**Python API:**
```python
from scripts.todoist_manager import TodoistManager

manager = TodoistManager()
manager.create_task(
    content="Morning workout",
    project="fitness",    # Maps to "work-out"
    priority="high",      # P1 in Todoist
    due_days=0,           # Today
    labels=["habit"]      # Add habit label
)
```

### Batch Task Creation

```python
tasks = [
    {"name": "Task 1", "project": "career", "priority": "high"},
    {"name": "Task 2", "project": "english", "priority": "medium"}
]
manager.create_tasks_batch(tasks)
```

### Goal Setup

```bash
# All three goals at once
python3 scripts/setup_goals.py --all

# Individual goals
python3 scripts/setup_goals.py --goal fitness
python3 scripts/setup_goals.py --goal career
python3 scripts/setup_goals.py --goal english
```

### Data Export

```bash
# Export to JSON
./lifeos export-tasks

# File location: ~/LifeOS/data/todoist_export_YYYYMMDD_HHMMSS.json
```

### Eudic Vocabulary Sync

**Setup:**
1. Get Eudic API Token from https://my.eudic.net/OpenAPI/Authorization
2. Update `config/eudic_config.json` with your token
3. (Optional) Add `EUDIC_TOKEN` to GitHub Secrets for automation

**Manual Sync:**
```bash
# Test connection (dry-run)
./lifeos test-eudic

# Small batch test (5 words)
./lifeos sync-eudic --limit 5

# Full sync (all vocabulary)
./lifeos sync-eudic
```

**GitHub Actions Setup:**
1. Go to repository Settings → Secrets and variables → Actions
2. Add secret: `EUDIC_TOKEN` = your API token
3. Workflow runs daily at 05:00 UTC (automatic)

**Workflow:**
```
欧路词典生词本 → Eudic API → Notion Anki Cards → sync-anki → Telegram (.apkg)
```

---

## Testing

```bash
# Test Todoist connection
./lifeos test

# Should output:
# ✅ 连接成功！
#    项目数: 4
#    标签数: 4
#    任务数: X
```

---

## Development Guidelines

### For Claude Code

1. **Task Management**:
   - Use TodoWrite to track multi-step tasks
   - Proactively send tasks to Todoist when appropriate
   - Prioritize action over explanation

2. **Project Tools First**:
   - Use `todoist_manager.py` for task operations
   - Use `personal_assistant.py` for NLP parsing
   - Avoid external tools when project tools exist

3. **Todoist Integration**:
   - All task operations go through Todoist API
   - No email/AppleScript methods
   - Cross-platform compatibility is key

4. **Data Privacy**:
   - API tokens stored in `config/todoist_config.json`
   - Never commit tokens to git
   - Use `.gitignore` for sensitive data

---

## Priority Mapping

```
high   → 4 (P1 - Urgent)
medium → 2 (P2 - Normal)
low    → 1 (P3 - Low)
```

---

## Key Permissions

The system has permissions for:
- Python script execution
- Todoist API access (requires API Token)
- File operations in project and Logseq directories
- Git operations for version control
- Web searches for specific domains

---

## Important Features

- **Cross-Platform**: Works on Windows, Mac, iPhone, Android, Web
- **API Integration**: Full programmatic control via Python
- **Data Export**: Export tasks to JSON for analysis
- **Goal Templates**: Pre-configured for fitness, career, English
- **Habit Tracking**: Built-in support for recurring tasks
- **Natural Language**: Parse user descriptions into structured tasks
- **Life Tracking**: Integrate with Logseq for journaling

---

## Documentation

- **Getting Help**: `./lifeos help`
- **Anki Sync Guide**: See `docs/ANKI_SYNC_QUICKSTART.md`
- **Anki Settings**: See `docs/ANKI_SETTINGS_FINAL.md`
- **Todoist API**: https://developer.todoist.com/rest/v2/
- **Notion API**: https://developers.notion.com/
- **Eudic OpenAPI**: https://my.eudic.net/OpenAPI/Doc_Index
- **Todoist SDK**: https://github.com/Doist/todoist-api-python

---

**Last Updated**: 2026-01-09

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/huangf06) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
