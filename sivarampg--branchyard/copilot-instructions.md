## branchyard

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

branchyard is a CLI tool for managing Git worktrees with parallel development workflows. It wraps `git worktree` commands with safety checks, session management, and multi-editor support.

## Technology Stack

- **Runtime**: Bun (v1.0+) - Use Bun for all operations
- **Language**: TypeScript with ES modules
- **Dependencies**:
  - `zod` for config/session validation
  - `colorette` for terminal colors (imported but not actively used yet)
- **Node APIs**: Always use `node:` prefix for built-in modules (`node:fs`, `node:path`, `node:url`)

## Development Commands

```bash
# Install dependencies
bun install

# Run CLI locally during development
bun run dev [args]        # or: bun run src/index.ts [args]

# Link globally for testing
bun link                  # Makes 'branchyard' command available globally
bun unlink               # Remove global link

# Publish to npm
bun run release          # or: bun publish --access public
```

## Architecture

### Command Flow

1. `src/index.ts` - Entry point, routes to commands based on arguments
2. `src/commands/*.ts` - Command implementations (create, remove, list, prune, interactive, config)
3. `src/utils/*.ts` - Shared utilities for git operations, prompts, config, sessions

### Key Architectural Decisions

**Session Management**: Sessions are stored in `~/.branchyard-sessions.json` with Zod validation. Each session stores worktree names, base branch, and editor preference.

**Config Storage**: User config in `~/.branchyardrc` (JSON) with Zod validation. Stores default editor and workspace templates.

**Parallel Operations**: Commands support both parallel (default) and sequential (`--sequential`) execution using `Promise.all()` for bulk operations.

**Safety Mechanisms**:

- Pre-flight checks verify Git/Bun installation and repo status
- Dry-run mode (`--dry-run`) for preview
- Force mode requires double confirmation
- Config/session files are validated with Zod schemas to prevent corruption

**Interactive Prompts**: Custom implementation using Bun's stdin stream reader (not using external prompt libraries).

### Easter Egg

The Oprah ASCII art in `src/assets/oprah.txt` displays when creating multiple worktrees. Keep this fun feature intact.

## Important Patterns

**Git Operations**: All Git commands use Bun's `$` template literal for shell execution:

```ts
await $`git worktree add ${dir} -b ${branch} ${base}`
```

**File I/O**: Use Bun's built-in functions where possible:

- `writeFileSync` from Bun for writing files
- Standard `node:fs` for other operations

**Error Handling**: Commands call `process.exit(1)` on errors rather than throwing exceptions.

## Testing Approach

Currently no automated tests. Manual testing checklist:

- `branchyard --help` - Verify help output
- `branchyard --fun` - Test Easter egg
- `branchyard list` - List worktrees
- Create/remove operations require a Git repository

## Code Style Conventions

- Use `node:` prefix for all Node.js built-in imports
- Async/await over promises
- Early returns for error conditions
- Direct process.exit() for CLI errors
- Template literals for multi-line strings

---
> Source: [SivaramPg/branchyard](https://github.com/SivaramPg/branchyard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-08 -->
