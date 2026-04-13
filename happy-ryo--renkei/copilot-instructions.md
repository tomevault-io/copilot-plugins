## renkei

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Communication Language / コミュニケーション言語

Please communicate in Japanese (日本語) when interacting with users in this repository. The codebase and documentation support both Japanese and English, but user interactions should be conducted in Japanese.

## Project Overview

Renkei (連携) is an AI-powered development assistant that integrates with tmux and Claude Code SDK. It orchestrates between an AI Manager and Claude Code to automate software development tasks through natural language instructions.

## Essential Commands

### Development
```bash
npm run dev          # Start in development mode with auto-reload
npm run build        # Compile TypeScript to JavaScript
npm run watch        # Watch mode for continuous development
```

### Testing & Quality
```bash
npm test                    # Run all tests
npm run test:watch         # Run tests in watch mode
npm run test:coverage      # Generate coverage report
npm run lint               # Check code with ESLint
npm run lint:fix          # Auto-fix ESLint issues
npm run format             # Format code with Prettier
npm run type-check         # Validate TypeScript types
npm run check              # Run all quality checks (type-check, lint, format, coverage)
```

### System Control
```bash
./scripts/renkei-setup     # Initial system setup
./scripts/renkei-start     # Start the Renkei system
./scripts/renkei-stop      # Stop the system
./scripts/renkei-task      # Execute a task
```

## Architecture Overview

### Core Components

1. **Managers** (`src/managers/`)
   - `AIManager`: Orchestrates AI task analysis and execution
   - `SessionManager`: Manages tmux sessions and persistence
   - `ConfigManager`: Handles system configuration and profiles
   - `TaskManager`: Coordinates task execution lifecycle

2. **Integrations** (`src/integrations/`)
   - `claude/`: Claude API integration for AI capabilities
   - Implements retry logic and error handling

3. **UI Layer** (`src/ui/`)
   - `TmuxManager`: Controls tmux pane layout and interactions
   - Real-time progress display and status monitoring

4. **Evaluators** (`src/evaluators/`)
   - Quality assessment for task completion
   - Continuous improvement cycle management

### Key Design Patterns

- **Manager Pattern**: Centralized control for major subsystems
- **Factory Pattern**: Used in `createClaudeIntegration`, `createSettingsManager`
- **Event-Driven**: Asynchronous communication between components
- **Permission System**: Sandbox execution with tool whitelisting/blacklisting

### Important Files & Interfaces

- `src/index.ts`: Main entry point and CLI command setup
- `src/interfaces/`: TypeScript type definitions for all major components
- `config/default-settings.json`: Default system configuration
- `src/utils/permissionValidator.ts`: Security enforcement for Claude Code tools

## Permission System

The system enforces strict permissions for Claude Code operations:
- **Allowed Tools**: Write, Read, Edit, MultiEdit, specific Bash commands
- **Denied Tools**: rm -rf, sudo, chmod, web access commands
- **Workspace Isolation**: Operations confined to `./workspace/` directory

## Testing Guidelines

- Tests are located alongside source files in `__tests__` directories
- Mock implementations available for all managers
- Minimum coverage threshold: 50%
- Use `npm run test:watch` during development for faster feedback

## Configuration

The system supports multiple configuration profiles:
- **development**: Local development with verbose logging
- **production**: Optimized for performance
- **custom**: User-defined configurations

Configuration hierarchy:
1. `config/default-settings.json` (base)
2. Profile-specific settings
3. User overrides via CLI flags

## Key Dependencies

- `@anthropic-ai/claude-code`: Claude Code SDK integration
- `commander`: CLI framework
- `inquirer`: Interactive prompts
- TypeScript 5.0+ with strict mode enabled
- Node.js 18+ required

## Development Guidelines

### Branch Management

When starting new work (bug fixes, features, or any modifications), always follow these steps:

1. **Create a new branch**
   - Always create a new branch from main before starting work
   - Use descriptive branch names following these conventions:
     - Bug fixes: `fix/issue-description`
     - New features: `feat/feature-name`
     - Documentation: `docs/update-description`
     - Refactoring: `refactor/component-name`

2. **Implement changes**
   - Work on your branch and commit changes appropriately
   - Write clear commit messages describing the changes

3. **CRITICAL: Check for uncommitted changes before creating PR**
   - **Always run `git status` to check for uncommitted changes**
   - **Review ALL modified files with `git diff` before committing**
   - **Never create a PR without verifying all intended changes are committed**
   - If formatter or linter made changes during commit hooks, review and include them

4. **Create Pull Request**
   - After completing work, create a PR if needed
   - Ensure review before merging to main branch

Example:
```bash
git checkout -b fix/tmux-pane-error
# Make your changes
git add .
# CRITICAL: Check what will be committed
git status
git diff --cached
# Review any unstaged changes
git diff
# If there are unstaged changes, decide whether to include them
git commit -m "fix: resolve tmux pane identification issue"
git push origin fix/tmux-pane-error
```

### Pre-PR Checklist
Before creating any PR, ALWAYS:
1. Run `git status` - ensure no untracked or modified files remain
2. Run `git diff` - review any unstaged changes
3. Run `git diff --cached` - review staged changes
4. Ensure all intended changes are included in commits
5. Check if formatters/linters modified files during commit

### Post-Merge Workflow
When the user informs you that a PR has been merged:
1. **Immediately switch to main branch**: `git checkout main`
2. **Pull the latest changes**: `git pull origin main`
3. **Verify the merge**: Check that all expected changes are present
4. **Clean up**: Delete the local feature branch if no longer needed

Example:
```bash
# After user says "マージしました" or "merged"
git checkout main
git pull origin main
git branch -d feature-branch-name  # Optional: clean up local branch
```

This ensures you're always working with the latest code and prevents conflicts.

## Common Development Tasks

### Adding a New Manager
1. Create interface in `src/interfaces/`
2. Implement manager in `src/managers/`
3. Add factory function if needed
4. Write comprehensive tests
5. Update `src/index.ts` if CLI integration needed

### Modifying Permission System
1. Edit `src/utils/permissionValidator.ts`
2. Update tests in `src/utils/__tests__/`
3. Document changes in permission system section

### Working with tmux Integration
- tmux 3.0+ required
- Pane layout managed by `TmuxManager`
- Session persistence handled automatically
- Use `tmux attach -t renkei` to manually inspect sessions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/happy-ryo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/happy-ryo)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
