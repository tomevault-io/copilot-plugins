## nodetsp

> This document provides guidelines for AI coding agents working on the nodetsp project.

# Agent Development Guidelines

This document provides guidelines for AI coding agents working on the nodetsp project.

## Project Overview

**nodetsp** is a CLI tool for scaffolding TypeScript projects with customizable configurations. It's organized as a **pnpm monorepo** with Turbo for build orchestration.

**Workspaces:**

- `cli/` - Main CLI tool (ESM only, built with tsdown)
- `docs/` - Next.js documentation site (not yet implemented)

## Build, Lint, and Test Commands

### Root Level (Monorepo)

```bash
# Install dependencies (enforces pnpm 10.26.2)
pnpm install

# Build all workspaces
pnpm build

# Run dev mode for all workspaces
pnpm dev

# Type checking across workspaces
pnpm typecheck

# Format all files with Prettier
pnpm format

# Lint all workspaces
pnpm lint

# Clean all build artifacts and node_modules
pnpm clean
```

### CLI Package (`cli/`)

```bash
# Build the CLI (tsdown + postbuild)
pnpm build

# Watch mode development
pnpm dev

# Type check without emitting
pnpm typecheck

# Format files
pnpm format

# Clean dist folder
pnpm clean

# Build and link globally for local testing
pnpm load

# Test the CLI after linking
nodetsp init
```

### Testing

**IMPORTANT:** No testing framework is currently configured. Do not attempt to run tests.

## Code Style Guidelines

### TypeScript Configuration

**Strict Mode Enabled:**

- `strict: true`
- `noUnusedLocals: true`
- `verbatimModuleSyntax: true`
- `isolatedModules: true`

**Module System:**

- Target: `esnext`
- Module: `preserve` (for bundler)
- Module Resolution: `bundler`
- Output: ESM only (`.mjs` extension)

### Imports

**Order and Style:**

1. Type imports first using `import type`
2. Node.js built-in modules (with `node:` prefix)
3. External dependencies
4. Internal imports using `@/` path alias

**Example:**

```typescript
import type { ProjectConfig } from "@/types";
import { mkdir } from "node:fs/promises";
import { join, dirname } from "node:path";
import { execSync } from "node:child_process";
import { copyTemplateFiles, getInstallCommand } from "../util";
```

**Rules:**

- Always use `node:` prefix for Node.js built-in modules
- Use `@/*` path alias for internal imports from `src/`
- Use `import type` for type-only imports
- Prefer named imports over default imports

### Formatting (Prettier)

**Configuration:**

- Semicolons: **required**
- Quotes: **double quotes**
- Trailing commas: **ES5 style** (objects, arrays)
- Print width: **80 characters**
- Tab width: **2 spaces**
- Arrow function parens: **always**
- Line endings: **LF**

**Run before committing:**

```bash
pnpm format
```

### Types and Naming Conventions

**File Naming:**

- Directories: lowercase (`lib`, `util`, `types`)
- TypeScript files: camelCase (`scaffold.ts`, `cli.ts`)
- Config files: kebab-case (`tsdown.config.ts`)

**Variable Naming:**

- camelCase for variables and functions
- PascalCase for types and interfaces
- SCREAMING_SNAKE_CASE for constants

**Type Definitions:**

- Centralize all types in `src/types/index.ts`
- Use `type` over `interface` unless extending
- Export all types that might be reused
- Use union types for literal options: `type PackageManager = "pnpm" | "npm" | "yarn";`

**Example:**

```typescript
export type PackageManager = "pnpm" | "npm" | "yarn";
export type ProjectConfig = {
  name: string;
  packageManager: PackageManager;
  folders?: Folders[];
  installDeps: boolean;
};
```

### Error Handling

**User Cancellation:**

```typescript
import { isCancel } from "@clack/prompts";

if (isCancel(value)) {
  process.exit(0); // Graceful exit
}
```

**Errors:**

- Use `process.exit(1)` for errors
- Provide clear error messages
- Handle file system errors gracefully

### File Documentation

**Header Comments:**
Every source file should include JSDoc comments:

```typescript
/**
 * @file scaffold.ts
 * @description Project scaffolding engine.
 * Responsible for creating the project structure, copying template files,
 * setting up optional directories, and performing initial setup tasks.
 */
```

### Code Organization

**Directory Structure:**

```
cli/src/
├── index.ts         # Entry point (commander setup)
├── lib/             # Core library code
│   ├── cli.ts       # Interactive prompts & validation
│   └── scaffold.ts  # Project scaffolding logic
├── types/           # Centralized type definitions
│   └── index.ts
└── util/            # Shared utilities
    └── index.ts
```

**Design Patterns:**

- **Separation of Concerns:** CLI logic separate from business logic
- **Barrel Exports:** Use `index.ts` files for clean imports
- **Pure Functions:** Prefer pure functions for utilities
- **Async/Await:** Use async/await over promises for readability

## Git and Commit Guidelines

### Branch Naming

- `feat/feature-name` - New features
- `fix/bug-description` - Bug fixes
- `docs/update-description` - Documentation
- `refactor/cleanup-description` - Refactoring

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

**Format:** `type(scope): description`

**Types:**

- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation changes
- `refactor` - Code refactoring
- `chore` - Maintenance tasks

**Examples:**

```
feat(cli): add new template option
fix(scaffold): resolve template copying issue
docs(readme): update installation instructions
```

### Pull Request Guidelines

**PR Titles:** Same format as commit messages

**PR Description Must Include:**

- Clear description of changes
- Problem statement (what issue you're solving)
- Testing steps
- Breaking changes (if any)

**Code Quality:**

- Small, focused commits (one logical change per commit)
- Follow existing patterns in the codebase
- Update documentation when needed
- No large monolithic commits

## Additional Notes

- **Package Manager:** Always use `pnpm` (enforced via packageManager field)
- **Build Tool:** CLI uses `tsdown` (modern TypeScript bundler)
- **CLI Framework:** Uses `commander` for command parsing
- **Prompts:** Uses `@clack/prompts` for interactive prompts
- **Colors:** Uses `picocolors` for terminal output
- **Template System:** Templates stored in `cli/templates/{tooling}/{module-system}/`
- **No ESLint:** CLI package relies on TypeScript strict mode

## Quick Reference

```bash
# Local development workflow
pnpm install          # Install dependencies
pnpm build            # Build all packages
pnpm load             # Build and link CLI globally
nodetsp init          # Test the CLI

# Before committing
pnpm format           # Format code
pnpm typecheck        # Check types
pnpm build            # Ensure builds succeed
```

## Resources

- Repository: https://github.com/devharshthakur/nodetsp
- License: MIT
- Author: Harsh Thakur <harshprojects2002@gmail.com>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/devharshthakur)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/devharshthakur)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
