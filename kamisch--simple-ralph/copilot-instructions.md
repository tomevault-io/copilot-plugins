## simple-ralph

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**simple-ralph** is a bash-based orchestration framework for AI coding agents. It implements the "Ralph Wiggum Technique" - a simple for-loop approach to incrementally complete tasks from a backlog (`prd.json`), with tight feedback loops and automatic verification.

The framework installs globally to `~/.ralph` and is invoked via the `simple-ralph` command.

## Architecture

```
simple-ralph (global install)
├── ~/.ralph/templates/     # Template files copied to projects
│   ├── ralph.sh           # Main orchestration loop
│   ├── setup_project.sh   # Project initialization
│   └── GENERATE_PRD_PROMPT.md
└── ~/.ralph/bin/
    └── simple-ralph       # CLI wrapper

Target project (after initialization)
├── plans/
│   ├── ralph.sh           # Orchestration script (copied)
│   ├── prd.json           # Task backlog (state)
│   └── progress.txt       # Execution log (memory)
└── ralph-post-hook.sh     # Optional custom verification
```

### Core Loop (`ralph.sh`)

1. Read incomplete tasks from `prd.json`
2. Send backlog + last 250 lines of `progress.txt` to AI agent (Docker sandbox)
3. Agent selects and implements highest-priority task
4. Agent outputs `COMPLETED_TASK_ID: task-X` when done
5. Run verification (`ralph-post-hook.sh` or auto-detected checks)
6. Mark task complete, append to progress, git commit
7. Repeat until all tasks done or max iterations reached
8. Output `promise complete here` when complete

### Key Files

- **prd.json**: JSON array with `id`, `description`, `passes`, `context` fields
- **progress.txt**: Append-only log (last 250 lines used as context)
- **ralph-post-hook.sh**: Custom verification script (overrides auto-detection)

## Commands

```bash
# Installation
./install_ralph.sh              # Install to ~/.ralph

# Initialize project
simple-ralph /path/to/project   # With PRD auto-generation
simple-ralph --no-generate .    # Template PRD only

# Run orchestration
./plans/ralph.sh                # Default 10 iterations
./plans/ralph.sh 20             # Custom max iterations

# PRD management
jq '.[] | select(.passes == false)' plans/prd.json  # View incomplete
jq '[.[] | select(.passes == false)] | length' plans/prd.json  # Count

# Verify installation
./verify_install.sh
```

## Docker Sandbox Authentication

Ralph requires Docker Desktop with AI Sandboxes. One-time authentication:

```bash
docker sandbox run claude  # Start interactive session
/login                     # Authenticate with Anthropic
# Exit with Ctrl+D
```

## Verification System

Order of precedence:
1. `ralph-post-hook.sh` (if exists and executable)
2. Auto-detection:
   - Node.js: `pnpm typecheck`, `pnpm test`, `pnpm lint`
   - Rust: `cargo test`, `cargo fmt --check`
   - Python: `pytest`

Verification failures stop execution and prevent task completion.

## Dependencies

**Required**: `bash`, `jq`, `git`, Docker Desktop (with AI Sandboxes), `expect`

**Optional**: pnpm (Node.js), cargo (Rust), pytest (Python)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kamisch)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kamisch)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
