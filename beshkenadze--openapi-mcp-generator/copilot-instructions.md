## openapi-mcp-generator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

Build all packages:
```bash
bun run build
```

Build specific package:
```bash
bunx turbo run build --filter=@aigentools/mcpgen-core
bunx turbo run build --filter=@aigentools/mcpgen
```

Type-checking:
```bash
bun run typecheck
```

Linting and formatting (Biome v2):
```bash
bun run lint
bun run format
```

Running tests:
```bash
bun run test
# Or for specific package:
bun test packages/core
```

Generate MCP server from OpenAPI spec:
```bash
# Build CLI first
bun run -w packages/cli build
# Generate server
bun run packages/cli -- --input ./examples/petstore.json --out ./servers/petstore --name petstore-mcp --runtime bun --force
```

Task runner commands (via Task):
```bash
# Generate Petstore server
task gen:petstore
# Run Prism mock server
task prism:mock
# Generate and inspect with MCP Inspector CLI
task prism:inspect:cli
# Generate and open MCP Inspector UI
task prism:inspect:ui
```

## Architecture

This is a Turborepo monorepo using Bun as package manager and runtime. It generates MCP (Model Context Protocol) servers from OpenAPI specifications.

### Core Packages

- **@aigentools/mcpgen-core** (`packages/core/`): Core generator library that parses OpenAPI specs and generates MCP server code
  - Entry: `src/index.ts`
  - Key exports: `generateServerFromOpenAPI`, `parseOpenApi`, `slugify`, `readTitleFromSpec`, `suggestNameFromSpec`
  - Generates single-file TypeScript MCP servers with Zod schema validation
  - Supports both JSON and YAML OpenAPI specs via @scalar/openapi-parser

- **@aigentools/mcpgen** (`packages/cli/`): Command-line wrapper for the core generator
  - Entry: `src/index.ts`
  - Interactive prompts using @clack/prompts
  - Parses CLI arguments for batch generation
  - Outputs Bun-optimized MCP server structure

### Generated Server Structure

The generator creates a single-file MCP server (`mcp-server/index.ts` or `src/server.ts`) that:
- Converts OpenAPI operations to MCP tools
- Uses Zod for input/output validation
- Includes inline `buildUrl` helper for path/query parameter handling
- Configures base URL via `API_BASE_URL` environment variable
- Exports individual tool definitions for testing

### Build System

- **Turborepo**: Manages build pipeline with dependency-aware task execution
- **TypeScript**: Project references for monorepo structure
- **Biome v2**: Linting and formatting
- **Bun bundler**: ESM builds with external dependencies preserved

### Testing Strategy

- Prism mock server for OpenAPI endpoint simulation
- MCP Inspector for validating generated servers
- Taskfile.yml contains smoke test workflows

## Release Management

For releasing packages and version management, see [docs/releasing.md](docs/releasing.md). The project uses Changesets for semver-compliant releases across the monorepo.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/beshkenadze)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/beshkenadze)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
