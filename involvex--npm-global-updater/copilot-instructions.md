## npm-global-updater

> **npm-global-updater** is a powerful TypeScript CLI tool for managing and updating globally installed npm packages. It abstracts common operations across multiple package managers (npm, pnpm, yarn, bun), providing a unified interface for developers.

# GEMINI.md - Project Context & Instructions

## Project Overview

**npm-global-updater** is a powerful TypeScript CLI tool for managing and updating globally installed npm packages. It abstracts common operations across multiple package managers (npm, pnpm, yarn, bun), providing a unified interface for developers.

- **Primary Runtime:** [Bun](https://bun.sh/) (used for development, testing, and building).
- **Core Technologies:** TypeScript, Bun, ESLint, Prettier.
- **Architecture:**
  - `src/index.ts`: Entry point for CLI argument parsing and command dispatching.
  - `src/commands/`: Individual command logic (e.g., `ls.ts`, `updateall.ts`, `export.ts`).
  - `src/utils/packageManager.ts`: Abstraction layer for `npm`, `pnpm`, `yarn`, and `bun` commands.
  - `src/config/`: Configuration management for the tool.
  - `src/notifications/` & `src/monitoring/`: Alert system for package updates.

## Building and Running

### Development

- **Run a command directly:** `bun run dev <command> [args]` (e.g., `bun run dev ls`).
- **Debug mode:** `bun run dev:debug <command>`.
- **Type checking:** `bun run typecheck`.

### Production Build

- **Build for Node.js:** `bun run build` (outputs to `bin/npm-updater.js`).
- **Build portable executable:** `bun run build:porteable` (outputs `bin/npm-updater.exe`).

### Testing & Linting

- **Run tests:** `bun test`.
- **Linting:** `bun run lint` (uses ESLint flat config).
- **Format code:** `bun run format` (uses Prettier).
- **Pre-build check:** `bun run prebuild` (runs format, lint:fix, and typecheck).

## Development Conventions

### Coding Style

- **TypeScript Strictness:** The project uses `strict: true`. Always provide proper types and avoid `any`.
- **Functional Commands:** Commands in `src/commands/` should be exported as async functions (e.g., `runls`, `runupdateall`).
- **Package Manager Abstraction:** Never execute raw PM commands directly. Use `getPackageManagerConfig` from `src/utils/packageManager.ts` to get the appropriate command strings for the user's selected manager.

### Contribution Workflow

1.  **Format and Lint:** Always run `bun run prebuild` before committing to ensure code quality.
2.  **Testing:** Add unit tests in the `tests/` directory for any new features or bug fixes. Use `bun test` to verify.
3.  **Command Registration:** New commands must be added to the `switch` statement in `src/index.ts` and documented in the `showHelp` function.

### File Naming

- Use `camelCase` for source files and directories (e.g., `packageTracker.ts`, `configManager.ts`).
- Command files in `src/commands/` should match the command name (e.g., `ls.ts`, `update.ts`).

## Key Dependencies

- `bun`: Runtime, test runner, and bundler.
- `typescript-eslint`: Linting for TypeScript.
- `console-clear`: Utility for clearing the terminal.
- `jiti`: Runtime TypeScript execution (where needed).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/involvex) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
