## openclaw-vikunja

> **Description:** Interact with Vikunja task management via REST API — create, query, and manage tasks and projects.

# Vikunja Skill

**Description:** Interact with Vikunja task management via REST API — create, query, and manage tasks and projects.

## API Endpoints

Vikunja exposes endpoints under `/api/v1/`:

### Projects
- `GET /api/v1/projects` — List all projects
- `GET /api/v1/projects/{id}` — Get project details
- `PUT /api/v1/projects/{id}/tasks` — Create task in project

### Tasks
- `GET /api/v1/projects/{id}/tasks` — List tasks in project
- `PUT /api/v1/tasks/{id}` — Update task
- `POST /api/v1/tasks/{id}` — Update task (alternative)
- `DELETE /api/v1/tasks/{id}` — Delete task

### Configuration

Required environment variables:
- `VIKUNJA_URL` — Vikunja instance URL (e.g., https://plan.agility.plus)
- `VIKUNJA_TOKEN` — API token with full privileges

## Usage Patterns

Natural language commands:
- "Create task: <title>" — Add new task to default project
- "Create task in <Project>: <title>" — Add task to specific project
- "List tasks in <Project>" — Show all tasks in project
- "List tasks" — Show tasks in all projects
- "Complete task <title>" — Mark task as done

## Task Properties

- `priority`: 1=Urgent 🔥, 2=High, 3=Medium, 4=Low, 5=Lowest, 6=None
- `due_date`: ISO 8601 date string
- `assignees`: Array of user IDs
- `labels`: Array of label names
- `description`: Task description

## Script Structure

```
vikunja/
├── SKILL.md           (this file)
├── README.md          # User documentation
├── api.js             # API client wrapper
├── commands.js        # Natural language parsing
└── index.js           # Main entry point
```

## Development Notes

- Vikunja uses **PUT** to create tasks (not POST)
- Task identifiers include project prefix (e.g., #1, #2)
- Default project is stored in user settings (`default_project_id`)
- Tasks can have multiple assignees and labels

---
> Source: [vernonstinebaker/openclaw-vikunja](https://github.com/vernonstinebaker/openclaw-vikunja) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
