## substrate

> Manages events, meetings, and scheduling with email invites/reminders via Resend.

# Substrate Technical Guide

Personal business operating system framework. Build business systems (CRM, tasks, calendar, notes) with PostgreSQL, Python, Absurd workflows, and AI-native interfaces.

**See [docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md) for full architecture documentation.**

## Database Schema

### notes.notes
Primary table for synced Obsidian vault content.

```sql
CREATE TABLE notes.notes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    file_path TEXT UNIQUE,           -- e.g. "Areas/CRM/Contacts/John.md"
    file_hash TEXT,                  -- SHA256 for change detection
    title TEXT,                      -- From frontmatter or filename
    content TEXT,                    -- Markdown body (no frontmatter)
    frontmatter JSONB DEFAULT '{}',  -- YAML metadata as JSON
    tags TEXT[] DEFAULT '{}',        -- e.g. ['#customer', '#sales']
    created_at TIMESTAMPTZ DEFAULT now(),
    updated_at TIMESTAMPTZ DEFAULT now()
);
```

**Query examples:**
```sql
-- Find by tag
SELECT title, file_path FROM notes.notes WHERE '#customer' = ANY(tags);

-- Search frontmatter
SELECT title FROM notes.notes WHERE frontmatter->>'status' = 'active';

-- Full text search
SELECT title FROM notes.notes WHERE content ILIKE '%product%';

-- Recent notes
SELECT title, updated_at FROM notes.notes ORDER BY updated_at DESC LIMIT 10;
```

### Frontmatter conventions
```yaml
---
status: draft | active | stable | archived
type: note | task | contact | meeting | project
area: sales | product | operations
tags: [customer, priority]
---
```

## Structure

```
.
├── substrate/              # Main codebase
│   ├── core/               # DB, worker, config (infrastructure)
│   ├── integrations/       # Obsidian, HubSpot, etc. (external data)
│   ├── domains/            # CRM, sales, notes (business logic)
│   └── ui/                 # API, chat, web, CLI (interfaces)
├── config/
│   └── sql/                # External SQL schemas
│       └── absurd.sql      # Absurd workflow engine schema
├── libs/
│   └── absurd-sdk/         # Vendored Absurd Python SDK
├── habitat/                # Task monitoring dashboard
├── .secrets/               # Deploy keys, credentials (gitignored)
├── vault/                  # Obsidian vault (symlink)
├── docker-compose.yml
└── pyproject.toml          # Dependencies (absurd-sdk from libs/)
```

## Commands

```bash
# Start all services
docker compose up -d

# Run migrations (after postgres is up)
python -m substrate.core.db.migrate

# Initialize absurd queue
PGPASSWORD=postgres psql -h localhost -U postgres -d substrate -c "SELECT absurd.create_queue('default')"

# Start worker locally (dev)
python -m substrate.core.worker.main

# Start API locally (dev)
uvicorn substrate.ui.api.main:app --reload

# View task dashboard
open http://localhost:7890
```

## Adding a Domain

```bash
mkdir -p substrate/domains/{name}/sql
```

Create:
- `sql/001_init.sql` - schema (`{name}.*` tables)
- `logic.py` - business rules
- `tasks.py` - worker tasks (optional)
- `tools.py` - AI tools (optional)

Then: `python -m substrate.core.db.migrate`

## Adding an Integration

```bash
mkdir substrate/integrations/{name}
```

Create:
- `sync.py` - fetch/push logic
- `tasks.py` - background sync tasks

## Conventions

| Item | Format | Example |
|------|--------|---------|
| Schema | `{domain}.*` | `crm.contacts` |
| Task | `{domain}.{action}` | `crm.enrich` |
| Migration | `NNN_*.sql` | `001_init.sql` |

## Database

```bash
PGPASSWORD=postgres psql -h localhost -U postgres -d substrate
```

## CRUD Web UI

**URL:** http://localhost:8000

A dark-themed web UI for browsing and editing all domain tables.

### Features
- Browse schemas (domains) and tables in sidebar
- View rows with pagination
- View full row details (expands JSON fields)
- Create new rows via JSON editor
- Edit existing rows
- Delete rows

### API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/domains` | List all domains |
| GET | `/api/{schema}/tables` | List tables in schema |
| GET | `/api/{schema}/{table}` | List rows (supports `?limit=N&offset=M`) |
| GET | `/api/{schema}/{table}/{id}` | Get single row |
| POST | `/api/{schema}/{table}` | Create row (JSON body) |
| PATCH | `/api/{schema}/{table}/{id}` | Update row (JSON body) |
| DELETE | `/api/{schema}/{table}/{id}` | Delete row |

### Usage Pattern

```python
# Query via API
import requests

# List notes
res = requests.get("http://localhost:8000/api/notes/notes?limit=10")
notes = res.json()["rows"]

# Create note
res = requests.post("http://localhost:8000/api/notes/notes", json={
    "title": "New Note",
    "content": "# Hello\n\nContent here",
    "tags": ["#work"]
})

# Update note
res = requests.patch(f"http://localhost:8000/api/notes/notes/{note_id}", json={
    "content": "Updated content"
})
```

## Two-Way Sync

The system supports bidirectional sync between PostgreSQL and the Obsidian vault.

### Flow: Vault → DB (read)
```
GitHub repo → git pull → vault/ → parse markdown → INSERT/UPDATE notes.notes
```

### Flow: DB → Vault (write)
```
create_note()/update_note() → INSERT/UPDATE DB → write .md file → git commit → git push
```

### AI Tools (substrate/domains/notes/tools.py)

| Tool | Description |
|------|-------------|
| `query_notes(query, tag, limit)` | Search notes by text or tag |
| `get_note(note_id)` | Get full note by UUID |
| `create_note(title, content, tags, frontmatter, folder)` | Create note in DB + vault + push to GitHub |
| `update_note(note_id, title, content, tags, frontmatter)` | Update note in DB + vault + push to GitHub |
| `delete_note(note_id)` | Delete from DB (file stays in vault) |

### Git Operations (substrate/integrations/obsidian/git.py)

| Function | Description |
|----------|-------------|
| `git_pull()` | Pull latest from remote |
| `git_status()` | Get changed files |
| `git_commit(message)` | Stage all + commit |
| `git_push()` | Push to remote |
| `git_commit_and_push(message)` | Atomic commit + push |

### Docker SSH Setup

Deploy key mounted at `/root/.ssh/id_ed25519`. Worker Dockerfile:
```dockerfile
RUN apt-get install -y git openssh-client
RUN ssh-keyscan github.com >> /root/.ssh/known_hosts
RUN git config --global user.email "substrate@bot.local"
```

## Calendar Domain

Manages events, meetings, and scheduling with email invites/reminders via Resend.

### Schema

```sql
-- Events table
calendar.events (
    id UUID PRIMARY KEY,
    title TEXT NOT NULL,
    description TEXT,
    location TEXT,                     -- physical or video link
    type TEXT DEFAULT 'meeting',       -- meeting, call, reminder, focus, appointment
    status TEXT DEFAULT 'confirmed',   -- tentative, confirmed, cancelled
    starts_at TIMESTAMPTZ NOT NULL,
    ends_at TIMESTAMPTZ NOT NULL,
    all_day BOOLEAN DEFAULT FALSE,
    contact_id UUID,                   -- link to crm.contacts
    company_id UUID,                   -- link to crm.companies
    remind_before_minutes INT,         -- e.g., 15, 60, 1440
    reminder_sent BOOLEAN DEFAULT FALSE,
    tags TEXT[] DEFAULT '{}',
    data JSONB DEFAULT '{}'            -- video_link, notes, etc.
)

-- Attendees table
calendar.attendees (
    id UUID PRIMARY KEY,
    event_id UUID NOT NULL,
    contact_id UUID,                   -- optional link to CRM
    email TEXT NOT NULL,
    name TEXT,
    status TEXT DEFAULT 'pending',     -- pending, accepted, declined, tentative
    is_organizer BOOLEAN DEFAULT FALSE,
    is_optional BOOLEAN DEFAULT FALSE,
    invite_sent BOOLEAN DEFAULT FALSE
)
```

### Views

| View | Description |
|------|-------------|
| `calendar.upcoming` | Future events, ordered by start time |
| `calendar.today` | Today's events |
| `calendar.needs_reminder` | Events needing reminder emails |

### Email Integration

**Outbound (via Resend):**
- Meeting invites with .ics attachment
- Reminder emails (configurable minutes before)
- Cancellation/update notifications

**Inbound:**
- Parse email replies for accept/decline via events router
- Auto-update attendee status

### Worker Tasks

| Task | Description |
|------|-------------|
| `calendar.send_invite` | Send invite email to attendees |
| `calendar.send_reminders` | Process events needing reminders |
| `calendar.create_interaction` | Create CRM interaction after meeting |

### Usage Examples

```python
# Create a meeting
event = calendar_events_create(
    title="Sales call with Acme",
    starts_at="2024-01-15T14:00:00Z",
    ends_at="2024-01-15T15:00:00Z",
    type="call",
    contact_id="uuid-of-contact",
    remind_before_minutes=15,
    attendees=[
        {"email": "john@acme.com", "name": "John Doe"},
        {"email": "me@company.com", "name": "Me", "is_organizer": True}
    ]
)

# Get today's schedule
today = calendar_today()

# Get upcoming week
upcoming = calendar_upcoming(days=7)

# Update attendee response
calendar_attendees_update(
    event_id="uuid",
    email="john@acme.com",
    status="accepted"
)
```

## MCP Server for Claude Code

An MCP server exposes all Substrate tools to Claude Code. Configured in `.mcp.json`.

### Available Tools

| Domain | Tools |
|--------|-------|
| **Notes** | `notes_query`, `notes_get`, `notes_create`, `notes_update`, `notes_delete` |
| **CRM** | `crm_contacts_query`, `crm_contacts_get`, `crm_contacts_create`, `crm_contacts_update` |
| **Companies** | `crm_companies_query`, `crm_companies_create` |
| **Interactions** | `crm_interactions_log`, `crm_interactions_get` |
| **Tasks** | `tasks_pending`, `tasks_query`, `tasks_get`, `tasks_create`, `tasks_update`, `tasks_complete`, `tasks_cancel` |
| **Events** | `events_query`, `events_get`, `events_rules_list`, `events_rules_create`, `events_rules_update`, `events_rules_delete`, `events_reprocess`, `events_create_manual` |
| **Calendar** | `calendar_events_query`, `calendar_events_get`, `calendar_today`, `calendar_upcoming`, `calendar_events_create`, `calendar_events_update`, `calendar_events_cancel`, `calendar_events_delete`, `calendar_attendees_add`, `calendar_attendees_update`, `calendar_attendees_remove` |

### Manual Setup

```bash
# Add to Claude Code manually
claude mcp add substrate -- .venv/bin/python -m substrate.ui.mcp.server

# Or use project config (.mcp.json already configured)
claude mcp list  # Verify substrate server

# Check status in Claude Code
/mcp
```

### Run Standalone

```bash
.venv/bin/python -m substrate.ui.mcp.server
```

## Specialized Agents

Domain-specific agents are defined in `.claude/agents/`. Each agent has focused tools and instructions for its domain.

| Agent | File | Purpose |
|-------|------|---------|
| **Notes** | `notes.md` | Manage Obsidian vault notes |
| **CRM** | `crm.md` | Contacts, companies, interactions |
| **Tasks** | `tasks.md` | Task management and tracking |
| **Calendar** | `calendar.md` | Events, scheduling, attendees |
| **Events** | `events.md` | Event routing rules |
| **Email** | `email.md` | Send/receive email, manage inboxes |

### Agent Tools

| Agent | MCP Tools |
|-------|-----------|
| **Notes** | `notes_query`, `notes_get`, `notes_create`, `notes_update`, `notes_delete` |
| **CRM** | `crm_contacts_*`, `crm_companies_*`, `crm_interactions_*` |
| **Tasks** | `tasks_pending`, `tasks_query`, `tasks_get`, `tasks_create`, `tasks_update`, `tasks_complete`, `tasks_cancel` |
| **Calendar** | `calendar_events_*`, `calendar_today`, `calendar_upcoming`, `calendar_attendees_*` |
| **Events** | `events_query`, `events_get`, `events_rules_*`, `events_reprocess`, `events_create_manual` |
| **Email** | `email_send`, `email_list`, `email_get`, `email_reply`, `email_stats`, `inboxes_*`, `users_*` |

### Usage

When working with Substrate, prefer using the specialized agent for the domain:

```
"Search my notes for project ideas" → Notes Agent
"Add a new contact from Acme Corp" → CRM Agent
"What meetings do I have today?" → Calendar Agent
"Send a follow-up email to John" → Email Agent
"Create a task to review the proposal" → Tasks Agent
"Set up a rule to auto-tag investor emails" → Events Agent
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/codimusmaximus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
