## idlergear

> Instructions for Gemini CLI when working on this project.

# Gemini CLI - idlergear

Instructions for Gemini CLI when working on this project.

## IdlerGear Integration

This project uses **IdlerGear** for knowledge management - a structured system that replaces ad-hoc files like TODO.md and NOTES.md with a command-based API.

### CRITICAL: Session Start

**Run this command at the start of EVERY session:**
```bash
idlergear context
```

This provides:
- Project vision and goals
- Current plan (if any)
- Open tasks and priorities
- Recent notes and explorations
- Reference documentation

### Core Commands

**Task Management:**
```bash
idlergear task create "description"  # Create a new task
idlergear task list                  # View all open tasks
idlergear task close <id>            # Close completed task
idlergear task show <id>             # View task details
```

**Notes & Insights:**
```bash
idlergear note create "content"      # Capture quick thought
idlergear note list                  # View all notes
idlergear note promote <id> --to task  # Convert note to task
```

**Project Context:**
```bash
idlergear vision show                # View project vision
idlergear plan show                  # View current plan
idlergear reference list             # List documentation
idlergear search "query"             # Search all knowledge
```

**File Registry (Token-Efficient):**
```bash
idlergear file deprecate old.py --successor new.py  # Mark file as deprecated
idlergear file annotate file.py --description "..." # Annotate for search
idlergear file search --query "authentication"      # Search files (93% savings!)
idlergear file status file.py                       # Check file status
```

### FORBIDDEN Patterns

**NEVER create these files:**
- TODO.md, TASKS.md
- NOTES.md, SESSION_*.md, SCRATCH.md
- BACKLOG.md, FEATURE_IDEAS.md

**NEVER write these comments:**
- `# TODO: ...`
- `# FIXME: ...`
- `// HACK: ...`
- `/* NOTE: ... */`

**ALWAYS use IdlerGear commands instead:**
```bash
# Instead of: # TODO: Add validation
idlergear task create "Add validation" --label tech-debt

# Instead of: # NOTE: API requires auth
idlergear note create "API requires auth header"

# Instead of: # FIXME: Memory leak
idlergear task create "Fix memory leak in processor" --label bug
```

### MCP Server (146 Tools)

IdlerGear provides 146 MCP tools for direct integration with Gemini CLI:
```bash
# MCP server automatically registered via ~/.gemini/settings.json

# Use MCP tools directly:
# - idlergear_task_create
# - idlergear_context
# - idlergear_file_search
# - idlergear_session_start
# ... and 142 more
```

See [MCP Tools Reference](docs/mcp-tools.md) for complete list.

### Development Workflow

1. **Start session**: `idlergear context`
2. **Check open tasks**: `idlergear task list`
3. **Work on task**: Make changes, commit
4. **Capture insights**: `idlergear note create "..."`
5. **Create new tasks**: `idlergear task create "..."`
6. **Save session**: `idlergear session-save` (optional)

### Token Efficiency

IdlerGear provides massive token savings:
- **Context**: 97% savings (15K → 570 tokens via `idlergear context --mode minimal`)
- **File Search**: 93% savings (15K → 200 tokens via file annotations)
- **Task List**: 90% savings with `--preview` flag

### Multi-Agent Coordination

If multiple AI agents work on the project:
```bash
# Start daemon for coordination
idlergear daemon start

# All agents automatically receive:
# - Registry updates (deprecated files)
# - Task changes
# - Broadcast messages
```

### Best Practices

1. ✅ **Run `idlergear context` at session start** - Always get current state
2. ✅ **Search file annotations before grep** - Save 93% tokens
3. ✅ **Create tasks immediately** - Don't defer with TODO comments
4. ✅ **Link deprecated files to successors** - Help future AI agents
5. ✅ **Annotate files proactively** - Describe purpose, tags, components

### Troubleshooting

**"Command not found: idlergear"**
- Install: `pipx install idlergear`
- Or: `pip install idlergear`

**"IdlerGear not initialized"**
- Run: `idlergear init`
- Then: `idlergear install`

**"MCP server not connected"**
- Check: `~/.gemini/settings.json`
- MCP server should be auto-registered during `idlergear install --gemini`

### Documentation

- [Full Documentation](https://github.com/marctjones/idlergear)
- [File Registry Guide](docs/guides/file-registry.md)
- [MCP Tools Reference](docs/mcp-tools.md)
- [Roadmap](ROADMAP.md)

---

**Remember**: Use IdlerGear commands, not files. It's the command-based API for AI-assisted development.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marctjones)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/marctjones)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
