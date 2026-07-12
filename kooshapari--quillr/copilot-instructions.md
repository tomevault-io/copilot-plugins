## quillr

> - **Description**: Type-safe HTTP client for TypeScript with interceptors and retries

# AGENTS.md - Quillr

## Project Overview

- **Name**: Quillr
- **Description**: Type-safe HTTP client for TypeScript with interceptors and retries
- **Language**: TypeScript
- **Location**: Phenotype repos shelf

## Features

- **Type-safe**: Full TypeScript inference
- **Interceptors**: Transform requests/responses
- **Retry**: Automatic retry with backoff
- **Mocking**: Built-in test utilities

## Architecture

```
src/
├── client.ts       # Core client implementation
├── interceptors/   # Request/response interceptors
├── retry/          # Retry logic with backoff
├── mock/           # Test mocking utilities
└── types.ts        # TypeScript type definitions
```

## Development Commands

```bash
# Install dependencies
npm install

# Build
npm run build

# Test
npm test

# Type check
npm run check

# Lint
npm run lint

# Format
npm run format
```

## Agent Rules

### Project-Specific Rules

1. **TypeScript Standards**
   - Strict TypeScript mode
   - Full type coverage required
   - No `any` types

2. **Testing**
   - Vitest for unit tests
   - Property-based testing where applicable
   - Mock utilities for HTTP testing

3. **Code Style**
   - camelCase for functions/variables
   - PascalCase for types/interfaces
   - Curly braces required for all blocks

### Phenotype Org Standard Rules

1. **UTF-8 encoding** in all text files
2. **Worktree discipline**: canonical repo stays on `main`
3. **CI completeness**: fix all CI failures before merging
4. **Never commit** agent directories (`.claude/`, `.codex/`, `.cursor/`)

## Quality Standards

```bash
npm run build     # Build with TypeScript
npm run check     # TypeScript type checking
npm run lint      # ESLint
npm run format    # Prettier formatting
npm test          # Vitest tests
npm test -- --coverage  # With coverage
```

## Git Workflow

1. Create feature branch: `git checkout -b feat/new-feature`
2. Add TypeScript types first
3. Implement functionality
4. Add tests with mocking
5. Run linting and type checking
6. Create PR with description

---
> Source: [KooshaPari/Quillr](https://github.com/KooshaPari/Quillr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
