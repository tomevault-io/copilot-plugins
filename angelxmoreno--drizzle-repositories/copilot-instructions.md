## drizzle-repositories

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Drizzle Repository Codegen Framework - an NPM package that generates type-safe, extensible repository classes for Drizzle ORM projects supporting PostgreSQL, MySQL, and SQLite with dependency injection and customizable codegen.

## Development Commands

### Runtime and Package Management
- Use `bun <file>` instead of `node <file>` or `ts-node <file>`
- Use `bun install` instead of `npm install`
- Use `bun run <script>` instead of `npm run <script>`

### Main Application
- `bun src/index.ts` - Run the main application
- `bun --hot src/index.ts` - Run with hot reload during development

### Testing
- `bun test` - Run all tests
- `bun test --watch` - Run tests in watch mode
- `bun test <pattern>` - Run specific test files

### TypeScript
- TypeScript is configured with strict settings in `tsconfig.json`
- Use `bun --check` to run TypeScript checks
- Target is ESNext with module preservation for optimal Bun compatibility

## Architecture

### Three-Layer Repository Pattern
1. **BaseRepository** - Abstract base classes with common operations (shared, dialect-specific)
2. **Generated Repositories** - Auto-generated from schema files (ephemeral, always overwritten)  
3. **Custom Repositories** - Developer extensions (never overwritten)

### Project Structure (Planned)
```
src/
├── cli/                    # CLI commands and parsing
├── core/                   # Core business logic
│   ├── generators/         # Code generation logic
│   ├── parsers/           # Schema file parsing
│   └── templates/         # Template generation
├── repositories/          # Base repository classes
│   ├── base/             # Abstract base classes
│   ├── dialects/         # Dialect-specific implementations
│   └── types/            # Repository type definitions
├── container/            # DI container setup
├── config/              # Configuration management
└── types/               # Global type definitions
```

### Multi-Dialect Support
- **PostgreSQL**: Using `Bun.sql` for connections
- **MySQL**: Bun's built-in MySQL support
- **SQLite**: Using `bun:sqlite` 

### Dependency Injection
- TSyringe-based with sub-container isolation
- Factory functions as fallback for non-DI users
- Injectable `RepositoryConfig` service

## Code Generation Strategy

### Configuration File
The system uses `repository.codegen.config.ts` for configuration with dialect selection, schema paths, output directories, and generation options.

### CLI Interface (Planned)
- `npx drizzle-repo-codegen generate` - Generate repositories from schema
- `npx drizzle-repo-codegen init` - Initialize config file
- `npx drizzle-repo-codegen validate` - Validate existing repositories

### Template System
- Simple template literals (no external dependencies)
- Dialect-aware imports and methods
- Type-safe generation with proper TypeScript types

## Development Guidelines

### Bun-Specific APIs
- `Bun.file` for file operations instead of `node:fs`
- `Bun.$` for shell commands instead of external process libraries
- Bun automatically loads .env files
- Use built-in WebSocket instead of `ws` library

### Testing Strategy
- Use Bun's built-in test runner with `bun:test`
- Docker Compose setup for multi-database testing (PostgreSQL, MySQL)
- Unit tests for individual methods
- Integration tests for full database operations across all dialects
- CLI functionality testing

### TypeScript Configuration
- Strict type checking enabled
- Module preservation for Bun compatibility
- ESNext target with latest features
- JSX support configured for potential React usage

### Code Quality
- Follow existing patterns in the codebase
- Maintain type safety throughout the codebase
- Use proper JSDoc documentation for generated code
- Handle errors gracefully with informative messages

### Git Workflow
- Do not do git add until I review the code

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/angelxmoreno)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/angelxmoreno)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
