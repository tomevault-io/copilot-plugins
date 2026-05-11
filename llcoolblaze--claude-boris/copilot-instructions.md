## claude-boris

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Reference

```bash
# Boris Workflow (Recommended)
/boris <task>        # Full orchestrated workflow with planning
/session-start       # Load Memory Bank, orient to project
/session-end         # Save context for next session

# Verification & Quality
/verify-all          # Run tests, types, lint, build
/test-and-fix        # Fix failing tests iteratively
/security-scan       # Check for vulnerabilities

# Git Workflow
/commit-push-pr      # Full git workflow with PR
/quick-commit        # Fast local commit
/undo                # Revert last Claude change
/checkpoint [name]   # Create named save point
/rollback [target]   # Restore checkpoint

# Context Management
/context             # Check context usage
/memory-init         # Initialize Memory Bank
/mode [mode]         # Switch modes (architect/code/debug/review)

# Issue Tracking
/fix-issue <num>     # End-to-end issue resolution
/ci-loop             # Push, wait for CI, fix, repeat

# Documentation
/update-claude-md    # Learn from mistakes
/first-principles    # Break down complex problems
```

## Project Overview

**What**: [Describe your project in one line]
**Stack**: [e.g., TypeScript, React, Node.js, PostgreSQL]
**Purpose**: [Why this project exists]

## Development Commands

| Command | Description |
|---------|-------------|
| `npm run dev` | Start development server |
| `npm run build` | Production build |
| `npm test` | Run test suite |
| `npm run lint` | Run ESLint |
| `npm run typecheck` | TypeScript check |

## Project Structure

```
src/
├── components/     # UI components
├── lib/            # Shared utilities
├── api/            # API routes/handlers
└── types/          # TypeScript types
```

## Code Conventions

### Style
- TypeScript strict mode
- Prefer `const` over `let`
- Functions < 20 lines ideal
- Meaningful variable names

### Naming
- Components: `PascalCase`
- Utilities: `camelCase`
- Constants: `SCREAMING_SNAKE_CASE`
- Files: `kebab-case`

### Patterns
- One component per file
- Co-locate tests with source
- Use conventional commits

## Git Workflow

- Branch: `feature/`, `fix/`, `chore/`
- Commits: `feat:`, `fix:`, `docs:`, `refactor:`
- Always create PR, never push directly to main

## Testing

- Follow AAA: Arrange, Act, Assert
- Mock external dependencies
- Test edge cases and error handling

---

## Mistakes to Avoid

<!-- Add entries when Claude makes mistakes -->

### Example
**What happened**: [Description]
**Why it's wrong**: [Explanation]
**Do instead**: [Correct approach]

---

## Learned Patterns

<!-- Document patterns that work well -->

### Example
**Pattern**: [Name]
**When**: [Context]
**How**: [Implementation]

---

## Memory Bank

This project uses Claude Boris Memory Bank for persistent context.

**Files**: `.claude/memory/`
- `projectContext.md` - Project understanding
- `activeContext.md` - Current session state
- `progress.md` - Task tracking
- `decisionLog.md` - Architecture decisions
- `conventions.md` - Learned patterns
- `sessionHistory.md` - Session summaries

**Usage**:
- Start sessions with `/session-start`
- End sessions with `/session-end`
- Memory persists across Claude sessions

---

*Update this file whenever Claude should remember something important.*

---
> Source: [llcoolblaze/claude-boris](https://github.com/llcoolblaze/claude-boris) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
