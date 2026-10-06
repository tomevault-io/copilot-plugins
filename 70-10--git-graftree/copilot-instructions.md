## git-graftree

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Language and Process

- Communicate in Japanese during development
- Think in English for technical reasoning
- Follow TDD approach as advocated by t_wada
- Make frequent, incremental commits at each step
- Write Git commit messages in English

## Commands

### Development
- `bun test` - Run all tests using Bun test runner
- `bun test test/config.test.ts` - Run specific test file
- `bun run build` - Build CLI executable for Node.js distribution
- `bun run build:bun` - Build native Bun executable (development only)
- `bun run lint` - Run oxlint to check code quality

### Local Testing
```bash
# Build and test CLI locally
bun run build
./dist/git-graftree.js <branch-name>
```

### Package Distribution
- `npm pack` - Create distributable package
- `npm run prepublishOnly` - Automatically runs before publishing (builds CLI)

## Architecture

### Core Purpose
git-graftree extends `git worktree` to automatically copy/symlink ignored files (like `.env`, config files) from the main directory into new worktree directories, solving the common problem of missing configuration files in worktrees.

### CLI Framework
Built with `citty` CLI framework instead of Commander.js for modern TypeScript-first CLI development.

### Path Structure Integration
Designed for ghq-style repository management:
- **With ghq**: `${ghq.root}/worktrees/${repo}/${branch}`  
- **Without ghq**: `~/ghq/worktrees/${repo}/${branch}` (fallback)

### Module Architecture

**Core Workflow (`src/cli.ts`)**:
1. Load configuration (`.graftreerc`)
2. Create git worktree with ghq-style paths
3. Expand glob patterns for file inclusion
4. Filter excluded patterns
5. Copy/symlink files to worktree
6. Add patterns to `.git/info/exclude`

**Key Modules**:
- `src/config.ts` - Configuration loading with cascading (global → local)
- `src/worktree.ts` - Git worktree operations + ghq integration
- `src/pattern-matching.ts` - Glob pattern expansion and filtering
- `src/file-operations.ts` - File copy/symlink operations
- `src/git-exclude.ts` - Git exclude file management

### Configuration System
- **File**: `.graftreerc` (JSON format)
- **Locations**: Global (`~/.graftreerc`) and local (project root)
- **Schema**: `{ mode: "copy"|"symlink", include: string[], exclude?: string[] }`
- **Cascading**: DEFAULT_CONFIG < global < local

### Node.js Compatibility
All code is Node.js 20+ compatible, avoiding Bun-specific APIs:
- Uses `fs/promises` instead of `Bun.file()`
- Uses `execa` for shell commands instead of `Bun.$`
- Built with `--target=node` for npm distribution

### Testing Strategy
- **Framework**: Bun test with TDD approach
- **Structure**: Comprehensive integration tests with temporary directories
- **Coverage**: All core modules tested including error scenarios
- **ghq Integration**: Tests verify fallback behavior when ghq unavailable

### Auto-Branch Creation
Automatically creates new branches if they don't exist using `git worktree add -b`, eliminating the need for manual branch creation.

---
> Source: [70-10/git-graftree](https://github.com/70-10/git-graftree) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
