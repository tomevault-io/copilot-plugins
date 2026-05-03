## waycore

> When working on tasks, you MUST follow the file-based progress tracking system defined in `docs/ai_instructions/progress_tracking.md`.

# Waycore Cursor Rules

## Progress Tracking Workflow (MANDATORY)

When working on tasks, you MUST follow the file-based progress tracking system defined in `docs/ai_instructions/progress_tracking.md`.

### Directory Structure

```
progress/
├── TODO/                    # Tasks not yet started
│   ├── phase-XX/            # Sequential feature phases
│   ├── prod-phase-XX/       # Production deployment phases
│   ├── improvements/        # Enhancements to existing features
│   └── ideas/               # Future feature concepts
├── IN_PROGRESS/             # Tasks currently being worked on
├── COMPLETED/               # Finished tasks (organized by YYYY-MM)
│   └── 2025-12/
└── BLOCKED/                 # Tasks blocked by dependencies
```

### Task Workflow

#### Starting a Task

1. **Find the task file** in `progress/TODO/phase-XX/` or appropriate directory
2. **Use the start script**: `./scripts/task-start.sh TASK_ID` (e.g., `./scripts/task-start.sh 13.1`)
3. The script will:
   - Move the task file to `progress/IN_PROGRESS/`
   - Update `**Status**: TODO` to `**Status**: IN_PROGRESS`
   - Update `**Started**: Not started` to `**Started**: YYYY-MM-DD`

#### During Work

- Update the task file with progress notes in "Implementation Notes" section
- Check off acceptance criteria as completed
- Document decisions and issues

#### Completing a Task

1. **Verify all acceptance criteria are met**
2. **Use the complete script**: `./scripts/task-complete.sh TASK_ID` (e.g., `./scripts/task-complete.sh 13.1`)
3. The script will:
   - Update `**Status**: IN_PROGRESS` to `**Status**: COMPLETED`
   - Update `**Completed**: Not completed` to `**Completed**: YYYY-MM-DD`
   - Move the task file to `progress/COMPLETED/YYYY-MM/`

### NEVER Do These

- ❌ Move tasks directly without using the scripts
- ❌ Create custom folders like `DONE/` (use `COMPLETED/YYYY-MM/` only)
- ❌ Mark tasks complete without running validation
- ❌ Skip the IN_PROGRESS step when working on tasks
- ❌ Use emoji status markers like `✅ DONE` (use `COMPLETED` text only)
- ❌ Include time or effort estimates in task files (no "Estimated effort", "Time Tracking", or hour estimates)

---

## Git Operations (MANDATORY)

**The agent MUST NOT commit or push changes to the repository.**

- ❌ NEVER run `git commit`
- ❌ NEVER run `git push`
- ❌ NEVER run `git merge` or `git rebase`
- ❌ NEVER modify git history in any way

The user is responsible for reviewing changes and managing all git operations. The agent may only:
- ✅ Run `git status` to check the current state
- ✅ Run `git diff` to review changes
- ✅ Run `git log` to view history (read-only)

### Task Status Values

Only use these exact status values:
- `TODO` - Task not started
- `IN_PROGRESS` - Task actively being worked on
- `COMPLETED` - Task finished and validated
- `BLOCKED` - Task cannot proceed due to dependency

### File Naming Convention

| Category | Format | Example |
|----------|--------|---------|
| Phase | `{phase}.{task}-{description}.md` | `13.1-meshtastic-architecture.md` |
| Prod-Phase | `P-{phase}.{task}-{description}.md` | `P-0.1-production-compose.md` |
| Improvement | `imp-{n}-{description}.md` | `imp-1-notes-rich-text.md` |
| Idea | `idea-{n}-{description}.md` | `idea-1-save-coordinates.md` |

---

## Code Quality Standards

### Before Completing Any Task

Run these validation commands:

```bash
# Linting
poetry run ruff check device/

# Type checking (exclude UI)
poetry run mypy device/ --exclude 'device/apps/ui/'

# Formatting
poetry run black --check device/

# Tests
poetry run pytest -q
```

### Python Style

- Use type hints for all function parameters and return values
- Follow PEP 8 naming conventions
- Maximum line length: 100 characters
- Use `from __future__ import annotations` in all Python files

### Type Ignore Comments (CRITICAL)

When using `# type: ignore` comments, you MUST specify the exact error codes being suppressed. Generic ignores cause CI failures when mypy versions differ.

**❌ NEVER use bare type ignores:**
```python
@decorator()  # type: ignore
@decorator()  # type: ignore[misc]  # Wrong if misc isn't the actual error
```

**✅ ALWAYS specify exact error codes:**
```python
@decorator()  # type: ignore[no-untyped-call, untyped-decorator]
some_call()  # type: ignore[arg-type]
```

**Common mypy error codes in this project:**
| Error Code | When It Occurs |
|------------|----------------|
| `no-untyped-call` | Calling a function without type hints |
| `untyped-decorator` | Decorator makes function untyped |
| `arg-type` | Wrong argument type |
| `return-value` | Wrong return type |
| `assignment` | Incompatible types in assignment |
| `misc` | Miscellaneous errors (rarely the actual code) |

**How to find the correct error code:**
1. Run `poetry run mypy device/` locally
2. Look at the error message - the code is in brackets: `[error-code]`
3. Use that exact code in your type ignore comment

**MCP Server decorators specifically require:**
```python
@server.list_tools()  # type: ignore[no-untyped-call, untyped-decorator]
@server.call_tool()   # type: ignore[untyped-decorator]
```

### Pre-commit Validation (MANDATORY)

Before considering ANY code change complete, you MUST run all validation commands and fix any errors:

```bash
# 1. Check formatting (must pass)
poetry run black --check device/

# 2. Check linting (must pass)
poetry run ruff check device/

# 3. Check types (must pass - this catches type: ignore issues!)
poetry run mypy device/

# 4. Run tests (must pass)
poetry run pytest -q
```

**If mypy shows "Unused type: ignore comment" errors:**
- The error code in your `# type: ignore[code]` doesn't match the actual error
- Run mypy without the ignore to see the real error code
- Update the ignore comment with the correct code(s)

**If mypy shows errors "not covered by type: ignore comment":**
- You're missing error codes in your type ignore
- Add all the error codes shown in the mypy output

### QML Style

- Use `App.Theme` for all colors, fonts, and spacing
- Import components as `import "." as App`
- Use `SensorBridge` for backend data access
- Handle offline/mock modes gracefully

---

## Git Commit Messages

Format: `type: description`

Types:
- `feat:` - New feature
- `fix:` - Bug fix
- `refactor:` - Code refactoring
- `docs:` - Documentation changes
- `test:` - Test additions/changes
- `chore:` - Maintenance tasks

Example: `feat: add sensor registry with dynamic discovery`

---

## Architecture Reminders

- Backend services communicate via MQTT message bus
- UI connects to backend via Unix sockets (production) or HTTP (development)
- Mock drivers are in `device/drivers/mock/`
- Real drivers will be in `device/drivers/real/` (future)
- All sensor data flows through SensorRegistry

---

## Database Schema Documentation (MANDATORY)

When making ANY changes to database structure, you MUST update `docs/database-schema.md`:

### Trigger Conditions

Update the schema documentation when:
- ✅ Adding, removing, or modifying database tables
- ✅ Adding, removing, or modifying table columns
- ✅ Adding or modifying database indexes
- ✅ Creating new database files (e.g., a new domain database)
- ✅ Changing default values for any database field
- ✅ Modifying database class methods that affect schema

### Update Checklist

1. Update the "Last Updated" date at the top of `docs/database-schema.md`
2. Update the relevant database section (General, Mesh, or AI)
3. If adding a new table:
   - Add table description
   - Document all columns with types and descriptions
   - Document any indexes
4. If adding a new database:
   - Add to the "Database Architecture" table
   - Create a new section with full schema
   - Update the "Future Databases" section if applicable
5. Update code examples if method signatures changed

### Database Files

| Database | Location | Owner Service |
|----------|----------|---------------|
| `general.sqlite3` | `/app/data/` | data-logger |
| `mesh.sqlite3` | `/app/data/` | comms-bridge |
| `ai.sqlite3` | `/app/data/` | data-logger |

### Database Classes

| Class | File |
|-------|------|
| `GeneralDatabase` | `device/libs/database/general.py` |
| `MeshDatabase` | `device/libs/database/mesh.py` |
| `AIDatabase` | `device/libs/database/ai.py` |

---

## Software Manual Documentation (MANDATORY)

When making changes to user-facing features, you MUST update the corresponding manual page in `docs/manual/`:

### Trigger Conditions

Update manual documentation when:
- ✅ Adding new user-facing features
- ✅ Changing how existing features work
- ✅ Adding new apps or screens
- ✅ Modifying settings or options
- ✅ Changing keyboard shortcuts or gestures
- ✅ Fixing user-visible bugs (update troubleshooting)

### Update Checklist

1. Find or create the relevant page in `docs/manual/`
2. Update feature descriptions to match implementation
3. Add screenshots or examples if helpful
4. Update troubleshooting if relevant
5. Check cross-references to other pages

### Documentation Style

- Write for non-technical users
- Use step-by-step instructions
- Include tips and common issues
- Keep language simple and clear

### Manual Structure

| Directory | Content |
|-----------|---------|
| `docs/manual/README.md` | Manual index and navigation |
| `docs/manual/getting-started.md` | First-time setup |
| `docs/manual/apps/` | App-specific guides |
| `docs/manual/features/` | Feature documentation |
| `docs/manual/reference/` | Shortcuts, troubleshooting |

---

## Reference Documents

- Architecture: `docs/architecture/architecture.md`
- API Specs: `docs/api/` (auto-generated OpenAPI specs)
- Database Schema: `docs/database-schema.md`
- User Manual: `docs/manual/` (user-facing documentation)
- AI Model Guide: `docs/models/README.md`
- Implementation phases: `local_plan/08-implementation-phases.md`
- Progress tracking: `docs/ai_instructions/progress_tracking.md`
- Design system: `local_plan/10-design-system.md`

---
> Source: [dmitry-grechko/waycore](https://github.com/dmitry-grechko/waycore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
