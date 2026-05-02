## kortex-agent

> > Quick reference for AI coding assistants exploring this codebase.

# AI Agent Context

> Quick reference for AI coding assistants exploring this codebase.

## Project Overview

**Kortex Agent** is a personal AI assistant with multi-provider LLM support and a unique "Council LLM" architecture where multiple AI models collaborate to provide better answers.

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | React + Vite |
| Backend | Python Flask |
| AI Providers | Google Gemini, OpenAI, Anthropic, OpenRouter (DeepSeek) |
| Data | Local JSON files |

## Directory Structure

```
kortex-agent/
├── kortex/              # Core Python package
│   ├── ai/              # AI handlers
│   │   ├── handler.py   # Main chat handler
│   │   ├── council.py   # Elite Mode (3 raw LLMs)
│   │   ├── hive.py      # Hive Mode (6 personas)
│   │   ├── mega.py      # MEGA Mode (Elite + Hive)
│   │   ├── scribe.py    # Background data updater
│   │   └── providers.py # LLM client abstraction
│   ├── config.py        # Configuration loader
│   ├── data.py          # JSON data operations
│   ├── backup.py        # Backup/restore
│   └── tools.py         # Function calling tools
├── backend/             # Flask API
│   ├── app.py           # Entry point
│   ├── errors.py        # Error handlers & decorators
│   └── routes/          # Modular API routes
│       ├── chat.py      # /api/chat
│       ├── council.py   # /api/council, /api/hive, /api/mega
│       ├── history.py   # /api/conversations
│       ├── data.py      # /api/data
│       ├── config.py    # /api/config
│       └── backup.py    # /api/backup
├── frontend/src/        # React app
│   ├── components/      # UI components
│   └── services/api.js  # API client
├── scripts/             # Utility scripts
│   ├── bump_version.sh  # Version sync
│   └── security_check.sh # Bandit scan
└── data/                # User data (gitignored)
```

## Key Concepts

### 1. Elite Mode (`kortex/ai/council.py`)
- 3 top LLMs (Gemini, GPT-5, Claude) respond **without personas**
- Anonymous peer review
- Chairman synthesizes final answer

### 2. Hive Mode (`kortex/ai/hive.py`)
- 6 DeepSeek models with **specialized personas**:
  - Devil's Advocate, Pure Data, Health Guardian
  - Values Keeper, Pragmatic Executor, Financial Realist
- Uses OpenRouter API

### 3. MEGA Mode (`kortex/ai/mega.py`)
- Elite + Hive run in parallel
- Voting system determines winners
- Mega Chairman synthesizes both

### 4. Scribe (`kortex/ai/scribe.py`)
- Background agent analyzing every conversation
- Autonomously updates user data files
- Uses function calling to modify JSON

### 5. Function Calling (`kortex/tools.py`)
- Tools: `update_*`, `create_data_file`, `read_data_file`
- All file operations go through `kortex/data.py`

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/chat` | POST | Main chat |
| `/api/council` | POST | Elite mode |
| `/api/hive` | POST | Hive mode |
| `/api/mega` | POST | MEGA mode |
| `/api/config` | GET/POST | Settings |
| `/api/backup/*` | POST | Backup/restore |

## Configuration

- `config.json` - API keys, model preferences (gitignored)
- `.env` - Environment variables (gitignored)
- Settings modal in UI for configuration

## Data Files

Located in `data/` (auto-created on first run):
- `profile.json` - User identity
- `health.json` - Health metrics
- `values.json` - Core values
- `finance.json` - Budget
- `active_projects.json` - Projects
- `conversations/` - Chat history

## Common Tasks

### Adding a new AI mode
1. Create handler in `kortex/ai/`
2. Add route in `backend/routes/council.py` (or new file)
3. Register in `backend/routes/__init__.py`
4. Add API function in `frontend/src/services/api.js`
5. Create UI component in `frontend/src/components/`

### Modifying function calling tools
1. Edit `kortex/tools.py`
2. Tools are auto-registered via `TOOL_FUNCTIONS` dict

### Adding new data files
1. Add default structure to `DEFAULT_DATA` in `kortex/data.py`
2. Add example in `data.example/`

### Creating a Release
1. Update version: `./scripts/bump_version.sh 1.0.0-alphaX`
2. Update `CHANGELOG.md` with changes
3. Commit: `git add . && git commit -m "Release vX.X.X"`
4. Tag: `git tag vX.X.X`
5. Push: `git push origin main --tags`
6. GitHub Actions auto-creates Release (alpha/beta = pre-release)

---
> Source: [jessepesse/kortex-agent](https://github.com/jessepesse/kortex-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
