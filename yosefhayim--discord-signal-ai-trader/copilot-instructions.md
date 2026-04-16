## discord-signal-ai-trader

> > **Note:** This file works alongside `AGENTS.md` (universal AI agent instructions). AGENTS.md contains the core intelligence rules for reuse, code style, and bug understanding. This file contains Gemini CLI-specific features and integrations.

# Gemini CLI Instructions

> **Note:** This file works alongside `AGENTS.md` (universal AI agent instructions). AGENTS.md contains the core intelligence rules for reuse, code style, and bug understanding. This file contains Gemini CLI-specific features and integrations.

## Core Intelligence Rules

**Read `AGENTS.md` first** - It contains critical rules for:
- Anti-duplication (search before creating)
- Code style patterns  
- Bug understanding protocol
- Self-improvement guidelines

## Project-Specific Patterns

**Read `docs/PROJECT_RULES.md`** for project-specific:
- Technology stack choices
- Component patterns
- Service patterns
- Testing conventions

## MCP Configuration for Gemini CLI

Configure Task Master MCP server in `~/.gemini/settings.json`:

```json
{
  "mcpServers": {
    "task-master-ai": {
      "command": "npx",
      "args": ["-y", "task-master-ai"]
    }
  }
}
```

**Note:** API keys are configured via `task-master models --setup`, not in MCP configuration.

## Gemini CLI-Specific Features

### Session Management

Built-in session commands:

- `/chat` - Start new conversation while keeping context
- `/checkpoint save <name>` - Save session state
- `/checkpoint load <name>` - Resume saved session
- `/memory show` - View loaded context

Both `AGENTS.md` and `GEMINI.md` are auto-loaded on every Gemini CLI session.

### Headless Mode for Automation

Non-interactive mode for scripts:

```bash
# Simple text response
gemini -p "What's the next task?"

# JSON output for parsing
gemini -p "List all pending tasks" --output-format json

# Stream events for long operations
gemini -p "Expand all tasks" --output-format stream-json
```

### Token Usage Monitoring

```bash
# In Gemini CLI session
/stats

# Shows: token usage, API costs, request counts
```

### Google Search Grounding

Leverage built-in Google Search as an alternative to Perplexity research mode:
- Best practices research
- Library documentation
- Security vulnerability checks
- Implementation patterns

## Important Differences from Other Agents

### No Slash Commands
Gemini CLI does not support custom slash commands (unlike Claude Code). Use natural language instead.

### No Tool Allowlist
Security is managed at the MCP level, not via agent configuration.

### Session Persistence
Use `/checkpoint` instead of git worktrees for managing multiple work contexts.

### Configuration Files
- Global: `~/.gemini/settings.json`
- Project: `.gemini/settings.json`
- **Not**: `.mcp.json` (that's for Claude Code)

## Recommended Model Configuration

For Gemini CLI users:

```bash
# Set Gemini as primary model
task-master models --set-main gemini-2.0-flash-exp
task-master models --set-fallback gemini-1.5-flash

# Optional: Use Perplexity for research (or rely on Google Search)
task-master models --set-research perplexity-llama-3.1-sonar-large-128k-online
```

## Task Master Quick Reference

### Essential Commands

```bash
# Daily workflow
task-master list                                   # Show all tasks
task-master next                                   # Get next task to work on
task-master show <id>                              # View task details
task-master set-status --id=<id> --status=done     # Mark complete

# Task management
task-master expand --id=<id> --research            # Break into subtasks
task-master update-subtask --id=<id> --prompt="notes"  # Log progress

# Project setup
task-master init                                   # Initialize project
task-master parse-prd <file>                       # Generate tasks from PRD
```

### MCP Tools

```javascript
// Equivalent MCP tools (when MCP connected)
get_tasks            // = task-master list
next_task            // = task-master next
get_task             // = task-master show <id>
set_task_status      // = task-master set-status
expand_task          // = task-master expand
update_subtask       // = task-master update-subtask
```

## Progress Tracking (MANDATORY)

**Update `progress-project.md` after EVERY task completion.**

This file is the single source of truth for project state. See AGENTS.md for full protocol.

```
Before work:  Read progress-project.md → Update "In Progress"
After work:   Update "Completed Work" → Clear "In Progress" → Update other sections
```

## Your Role with Gemini CLI

As a Gemini CLI assistant with this project:

1. **Check AGENTS.md first** - Follow anti-duplication and reuse rules
2. **Check PROJECT_RULES.md** - Follow project-specific patterns
3. **Use MCP tools naturally** - They integrate transparently in conversation
4. **Update progress-project.md** - **MANDATORY** after every task (non-negotiable)
5. **Save checkpoints** - Offer to save state after significant progress
6. **Monitor usage** - Remind users about `/stats` for long sessions
7. **Use Google Search** - Leverage search grounding for research
8. **Update docs** - Improve PROJECT_RULES.md when you discover patterns

**Key Principle:** Focus on natural conversation while behaving like a senior engineer who knows this codebase and documents their work. Task Master MCP tools work seamlessly with Gemini CLI's interface.

---

*See AGENTS.md for complete anti-duplication rules, code style patterns, and bug understanding protocol.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/YosefHayim) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
