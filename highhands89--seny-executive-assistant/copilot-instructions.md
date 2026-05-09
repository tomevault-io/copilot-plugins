## seny-executive-assistant

> This project uses OpenWolf for context management. Read and follow .wolf/OPENWOLF.md every session. Check .wolf/cerebrum.md before generating code. Check .wolf/anatomy.md before reading files.

# OpenWolf

@.wolf/OPENWOLF.md

This project uses OpenWolf for context management. Read and follow .wolf/OPENWOLF.md every session. Check .wolf/cerebrum.md before generating code. Check .wolf/anatomy.md before reading files.


# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## What is Seny?

Seny is a personal AI assistant built as a "Second Brain" system. It uses Claude (Anthropic) as its AI backbone and integrates with Gmail, Google Calendar, Slack, Telegram, and more. Users deploy their own instance and personalize it through a setup wizard.

## Tech Stack

- **Backend:** Python 3.11+, FastAPI, Anthropic SDK
- **Database:** PostgreSQL (all environments)
- **Auth:** JWT (python-jose), bcrypt
- **Frontend:** React (TypeScript), Radix UI, Tailwind CSS
- **Build:** Vite
- **Deployment:** Railway (recommended) or Docker
- **Integrations:** Gmail, Google Calendar, Slack, Telegram, Microsoft Outlook

## Project Structure

```
/
├── CLAUDE.md               # This file
├── README.md               # User-facing documentation
├── Dockerfile              # Container build
├── start.sh                # Docker/Railway entrypoint
├── requirements.txt    # Python dependencies
├── web/                    # FastAPI application
│   ├── main.py             # App entry point, middleware, startup
│   ├── api/                # REST API endpoints
│   ├── services/           # Business logic (Claude, Gmail, Slack, etc.)
│   ├── core/               # Database, scheduler, auth utilities
│   ├── auth/               # JWT token management
│   └── frontend/           # React app (TypeScript + Vite)
│       ├── src/
│       │   ├── pages/      # Page components
│       │   ├── components/ # Reusable UI components
│       │   ├── contexts/   # React Context (auth, etc.)
│       │   └── lib/        # Utilities (API client, helpers)
│       └── package.json
├── src/                    # Shared utilities (Config, token management)
│   └── core/
├── agent/                  # Browser history sync agent (optional)
├── screen_agent/           # Desktop screen agent (optional)
├── scripts/                # Database migration tools
├── tests/                  # pytest test suite
└── docs/                   # Integration setup guides
```

## Development Setup

```bash
# Clone and setup
git clone <your-fork-url>
cd seny
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Install frontend dependencies
cd web/frontend
npm install
cd ../..

# Start PostgreSQL (requires Docker)
docker-compose up -d

# Create .env from template
cp .env.example .env
# Edit .env — add at minimum: ANTHROPIC_API_KEY and SECRET_KEY
# DATABASE_URL is pre-configured for docker-compose

# Run locally
python start.py
# Visit http://localhost:8000
```

## Code Patterns

### API Routes

Routes live in `web/api/`. Each integration has its own file (email.py, calendar.py, slack.py, etc.). All routes are registered in `web/main.py`.

### Services

Business logic lives in `web/services/`. The main service is `claude_service.py` which handles all Claude API interaction, system prompt building, and tool execution.

### Database

`web/core/database.py` handles all database operations. Schema is auto-created on first run via `init_db()`. Requires PostgreSQL (`DATABASE_URL` env var must be set).

### Authentication

JWT-based auth via `web/auth/jwt_utils.py`. Use `Depends(require_auth)` on FastAPI endpoints to require authentication. Returns `user_id` as string.

### Frontend

React app in `web/frontend/`. Uses Radix UI primitives with Tailwind CSS styling. Build with:

```bash
cd web/frontend && npm run build
```

Output goes to `web/static/react/`.

### System Prompt

The system prompt is built dynamically in `claude_service.py`. It loads user profile from `get_user_profile()` and injects name, pronouns, key people, projects, and priorities at runtime. No hardcoded personal data.

## Code Style

- **Simplicity first** — avoid over-engineering
- **Complete files** — not snippets requiring manual integration
- **Working on first run** — test before declaring done
- **No hardcoded secrets** — use environment variables

## Git Conventions

Commit format: `<type>(<scope>): <description>`

Types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`

## When to Ask Questions

- Feature conflicts with architecture
- Multiple valid approaches exist
- Scope is ambiguous
- Security tradeoffs involved

## Current Integrations

| Integration | Status |
|-------------|--------|
| Web Search | Built-in |
| Gmail | OAuth required |
| Google Calendar | OAuth required |
| Notes | Built-in |
| Tasks | Built-in |
| Slack | OAuth required |
| Telegram | API credentials required |
| Microsoft Outlook | OAuth required |
| Screen Agent | Optional desktop app |
| Browser History | Optional Python script |

---
> Source: [highhands89/seny-executive-assistant](https://github.com/highhands89/seny-executive-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
