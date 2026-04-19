## momo-assistant

> After completing any code change, bug fix, new feature, or planned task, you MUST update the Notion project tracker using the helper script:

# Momo Project Rules

## Notion Project Tracker — ALWAYS UPDATE

After completing any code change, bug fix, new feature, or planned task, you MUST update the Notion project tracker using the helper script:

```bash
# Add a new task when planning work — ALWAYS include --description
python scripts/update_notion_tracker.py add "Task description" --status "To Do" --priority "High" --component "Gmail" --type "Feature" --effort "Small" --description "Detailed explanation of what this task involves, why it's needed, and what files/components are affected."

# Mark a task as in progress when starting
python scripts/update_notion_tracker.py update "Task description" --status "In Progress"

# Mark a task as done when finished
python scripts/update_notion_tracker.py update "Task description" --status "Done"

# List current tasks
python scripts/update_notion_tracker.py list --status "In Progress"
```

### IMPORTANT: Always include --description
When creating a task, you MUST provide a `--description` with a clear explanation of:
- What the task involves
- Why it's needed
- Which files/components are affected
- Any relevant technical details

### When to update:
- **Starting work**: set task to "In Progress" (create it first if it doesn't exist)
- **Finishing work**: set task to "Done"
- **Planning work**: create task as "To Do" or "Backlog"
- **Hit a blocker**: set task to "Blocked"

### Valid values:
- Status: Backlog, To Do, In Progress, Done, Blocked
- Priority: Critical, High, Medium, Low
- Component: Briefing, Gmail, Calendar, Tasks, Knowledge Graph, Granola, Gemini, Chat, Proactive Intelligence, Infrastructure
- Type: Feature, Bug Fix, Performance, Refactor, DevOps
- Effort: Small, Medium, Large

### Notion Database ID: set via NOTION_DATABASE_ID env var in .env

## Project context
- Stack: Python, FastAPI, Gemini, Google Workspace APIs, Firestore, Cloud Run
- Config loads from .env via dotenv (config.py)
- Tiered model routing: Flash for standard, Pro for deep reasoning
- Knowledge graph in Firestore with entity extraction
- Granola MCP for meeting notes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/icekarim) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
