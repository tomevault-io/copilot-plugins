## c3

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a **Claude Code configuration harness** that provides reusable skills, agents, and settings for Python/Baseweb development projects. The configuration is designed to be symlinked into `~/.claude/` for use across multiple projects.

## Installation

```bash
make install
```

This symlinks the `agents/`, `skills/`, `bin/`, and `settings.json` into `~/.claude/`.

## Project Structure

```
c3/
├── agents/           # Specialized agent definitions
├── skills/           # Reusable skill definitions
├── bin/              # Utility scripts (statusline)
├── settings.json     # Claude Code configuration
└── Makefile          # Installation commands
```

## Skills Overview

Skills are invoked via `/skill-name`. See README.md for complete catalog.

| Category | Skills | Purpose |
|----------|--------|---------|
| **Project** | project, project-feature, project-manage, project-status | Feature intake and implementation workflow |
| **Domain** | python, pymongo, baseweb, fire, textual, rich | Framework/library expertise |
| **Development** | develop-skill, develop-agent | Create skills and agents |
| **Utility** | commit, bug-fixing, naming, git-activity-report, ... | Common workflows and tools |

## Agents Overview

Agents are specialized for structured project development. See README.md for complete catalog.

| Role | Agents |
|------|--------|
| **Analysis** | functional-analyst, researcher, api-architect |
| **Design** | ui-ux-designer |
| **Implementation** | python-developer |
| **Review** | code-reviewer, testing-engineer, security-engineer |
| **Documentation** | end-user-documenter |

## Project Management Workflow

The `/project` dispatcher routes to specialized skills:

- **`/project feature <description>`** — Capture new feature ideas
- **`/project status`** — Quick overview of project state
- **`/project manage`** — Full implementation workflow

### Implementation Workflow (via `/project-manage`)

1. **Functional Analysis** - functional-analyst reviews requirements and creates TODO.md
2. **Cross-Domain Review** - api-architect and ui-ux-designer provide perspective
3. **Consensus** - Agents agree on backlog before implementation
4. **Implementation Loop** - For each task: Plan → Implement → Review cycle

## Key Conventions

### Indentation
- **Always use two spaces** for indentation in all file types

### Testing Patterns (pytest)
- Use `monkeypatch` fixture for environment variables
- Use `unittest.mock.patch` and `MagicMock` for mocking
- Use `autouse=True` fixtures for test setup
- Group related tests in classes
- Test both success and error paths

### Database Security (MongoDB)
- Never log connection URIs (may contain credentials)
- Use `bson.errors.InvalidId` for ObjectId validation
- Escape regex input with `re.escape()` to prevent ReDoS
- Use environment variables for pool sizes

### API Endpoints (Baseweb/Flask)
- Use `endpoint="api.name"` to avoid route name collisions
- Use `@server.authenticated("permission.name")` decorator
- Catch specific exceptions before generic ones

### Error Handling
```python
try:
  # database operation
except NotFoundError:
  raise  # re-raise domain exceptions
except PyMongoError as e:
  logger.error(f"Database error: {e}\n{traceback.format_exc()}")
  raise DatabaseError(f"Failed: {e}")
```

## File Conventions

- `analysis/` - Functional, API, UI/UX analysis documents
- `reporting/{task-name}/` - Task-specific reports (plan.md, summary.md, review files)
- `TODO.md` - Backlog with phase-organized tasks
- `CLAUDE.md` - This file (project-specific guidance)
- `AGENTS.md` - Agent instructions for target projects

## Status Line

The status line script (`bin/statusline.py`) displays:
- Model name
- Context window usage percentage
- Session duration
- Rate limit percentages
- Current git branch

---
> Source: [christophevg/c3](https://github.com/christophevg/c3) — distributed by [TomeVault](https://tomevault.io/claim/christophevg).
<!-- tomevault:4.0:copilot_instructions:2026-04-18 -->
