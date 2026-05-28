## contextmesh-cli

> TypeScript CLI for ContextMesh - npm-like package manager for MCP connectors.

# ContextMesh CLI

## Overview
TypeScript CLI for ContextMesh - npm-like package manager for MCP connectors.

## Stack
- Node.js 18+
- TypeScript 5.3
- Commander.js for CLI
- Jest for testing

## Development

```bash
# Install dependencies
npm install

# Run in development
npm run dev publish ./test-connector

# Run tests
npm test

# Type checking
npm run typecheck

# Build
npm run build
```

## Project Structure
```
src/
├── cli.ts           # Main CLI entry point
├── commands/        # CLI commands
│   └── publish.ts   # Publish command implementation
├── utils/           # Utility functions
│   ├── manifest.ts  # Manifest handling
│   ├── validator.ts # Schema validation
│   └── publisher.ts # Publishing logic
└── types/           # TypeScript types
```

## Key Features
- Manifest validation with JSON Schema
- ZIP archive creation with file filtering
- SHA-256 checksum generation
- Progress indicators
- Dry-run mode

## Testing
- Unit tests for all utilities
- Integration tests for commands
- Mock registry API for testing

## CI/CD
- GitHub Actions for testing
- Automatic npm publishing on release

---
> Source: [cloudfloo/contextmesh-cli](https://github.com/cloudfloo/contextmesh-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
