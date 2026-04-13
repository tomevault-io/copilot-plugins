## amp-templates

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

TypeScript/JavaScript monorepo for AMP Templates - a CLI tool and template collection for scaffolding Amp-powered applications. Built with Effect-TS, pnpm workspaces, and modern TypeScript tooling.

## Architecture

### Monorepo Structure

```text
amp-templates/
├── packages/
│   └── create-amp/              # CLI tool (Effect-TS based)
├── templates/                   # Template files downloaded via GitHub
│   ├── backend/                 # Backend templates (Express, Fastify, Apollo GraphQL)
│   │   ├── base/               # Shared backend files
│   │   ├── express/            # Express + ArrowFlight (local)
│   │   ├── fastify/            # Fastify + ArrowFlight (local)
│   │   ├── apollo-graphql/     # Apollo GraphQL + ArrowFlight (local)
│   │   ├── express-gateway/    # Express + Gateway (remote)
│   │   ├── fastify-gateway/    # Fastify + Gateway (remote)
│   │   └── apollo-graphql-gateway/ # Apollo GraphQL + Gateway (remote)
│   ├── nextjs/                 # Next.js fullstack template
│   │   ├── base/               # Shared Next.js files
│   │   └── arrow-flight/       # ArrowFlight integration
│   └── vite-react/             # Vite + React templates
│       ├── base/               # Shared Vite files (includes docker-compose, contracts, amp.config.ts)
│       ├── ampsync-electricsql/ # AmSync + Electric SQL
│       ├── flight-atom/        # ArrowFlight + effect-atom
│       ├── jsonl-atom/         # JSON Lines + effect-atom
│       ├── jsonl-tanstack-query/ # JSON Lines + TanStack Query
│       └── jsonl-transfers/    # JSON Lines with existing Arbitrum dataset
└── examples/                    # Full example applications
    ├── backend/                # Backend examples
    ├── nextjs/                 # Next.js examples
    ├── nextjs-electricsql/     # Next.js + Electric SQL example
    └── vite-react/             # Vite + React examples
```

### Effect-TS Architecture

The CLI (`packages/create-amp`) is built entirely with Effect-TS:

- **Effect Services**: `GitHub.GitHubService` handles template downloads from GitHub tarball
- **Effect CLI**: Uses `@effect/cli` for command parsing, options, arguments, and prompts
- **Effect Platform**: Uses `@effect/platform` and `@effect/platform-node` for file system operations
- **Functional Composition**: All logic composed with Effect operators (`pipe`, `Effect.gen`, etc.)

Key modules:
- `src/Cli.ts` - Effect CLI command definition, handlers, and interactive prompts
- `src/GitHub.ts` - Effect Service for downloading templates from GitHub via HTTP streaming
- `src/internal/Templates.ts` - Template registry defining all available templates
- `src/Domain.ts` - Effect Schema definitions for type-safe domain models
- `src/Utils.ts` - Utility functions for path resolution and validation
- `src/bin.ts` - Entry point with Effect runtime setup
- `src/bun.ts` - Alternative entry for local development with tsx

### Template Download Mechanism

Templates are NOT bundled with the CLI package. Instead:
1. CLI downloads `https://codeload.github.com/edgeandnode/amp-templates/tar.gz/main`
2. Extracts only the specific template directory path (e.g., `/templates/backend/express`)
3. Strips parent directories and writes directly to target project directory
4. Post-processes `package.json` to set project name
5. Optionally runs `git init` and package manager install

### Template Types

Templates are categorized into two types:
- **"build-dataset"**: Includes local infrastructure (Docker Compose with Anvil, Ampd, etc.) for building custom datasets
- **"existing-dataset"**: Connects to remote Amp Gateway for querying existing datasets (no local infra)

## Commands Reference

### Development Workflow

```bash
# Install dependencies
pnpm install

# Build all packages (includes type checking)
pnpm build

# Run tests with Vitest
pnpm test

# Run tests with coverage
pnpm test:coverage

# Lint code
pnpm lint

# Auto-fix lint issues
pnpm lint:fix

# Format code with Prettier
pnpm format

# Clean build artifacts
pnpm clean
```

### CLI Development

```bash
# Build the CLI package
cd packages/create-amp
pnpm build

# Run CLI in development mode (uses tsx)
cd packages/create-amp
pnpm create-amp my-app

# Or with options
cd packages/create-amp
pnpm create-amp my-app --template backend-express --package-manager pnpm

# Available CLI options:
--template <key>              # Template key from AVAILABLE_TEMPLATES
--package-manager <pm>        # pnpm|npm|yarn|bun
--skip-install-deps          # Skip running package manager install
--skip-initialize-git        # Skip git init
```

### Just Commands

If you have `just` installed (justfile available):

```bash
just                    # List all commands
just build              # Build all packages
just test               # Run tests
just lint               # Lint code
just lint-fix           # Lint and fix
just fmt-ts             # Format TypeScript
just test-coverage      # Run tests with coverage
```

## Code Standards

### TypeScript Configuration

- **Base Config**: `tsconfig.base.jsonc`
  - Strict mode enabled with `exactOptionalPropertyTypes`
  - Target: ES2022
  - Module: ESNext with bundler resolution
  - Composite projects for monorepo performance
  - JSX: react-jsx

### Code Style

- **Prettier** (`.prettierrc`):
  - Print width: 120
  - No semicolons
  - Double quotes
  - Tailwind CSS plugin enabled
- **ESLint** (`eslint.config.mjs`):
  - TypeScript-ESLint
  - Import plugin with auto-ordering (alphabetical, groups)
  - Perfectionist plugin for sorted exports
  - Ignores: `templates/`, `examples/`, `dist/`, `.next/`
- **File type**: ES modules (`"type": "module"`)

### Testing

- **Framework**: Vitest with globals enabled
- **Config**: `vitest.config.ts` + `vitest.shared.ts`
- **Coverage**: V8 provider
- **Test files**: `test/**/*.{test,spec}.{ts,tsx,js,jsx}`

## Important Patterns

### Effect-TS Patterns

When working with the CLI codebase:

- Use `Effect.gen` for imperative-style effect composition
- Use `pipe` for functional composition chains
- Define errors with `Data.TaggedError`
- Services extend `Effect.Service()()` with accessors
- Use Effect Schema for runtime validation and type derivation
- Options/Arguments validated with Effect CLI primitives

Example from `Cli.ts`:
```typescript
const scaffoldAmpApp = Effect.fn("ScaffoldAmpApp")(function* (config: ResolvedConfig) {
  const fs = yield* FileSystem.FileSystem
  const path = yield* Path.Path

  yield* fs.makeDirectory(config.appName, { recursive: true })
  yield* GitHub.GitHubService.downloadTemplate(config.appName, template)
  // ...
})
```

### Adding New Templates

To add a new template:

1. Create template directory under `templates/backend/`, `templates/nextjs/`, or `templates/vite-react/`
2. Add entry to `AVAILABLE_TEMPLATES` in `packages/create-amp/src/internal/Templates.ts`:
   ```typescript
   "template-key": {
     key: "template-key",
     name: "Display Name",
     description: "Description shown in prompts",
     directory: "/templates/path/to/template",
     skip: new Set([...Constants.ALWAYS_SKIP_DIRECTORIES, "dist"]),
     type: "build-dataset" // or "existing-dataset"
   }
   ```
3. Add key to `AvailableTemplFrameworkKey` literal in `packages/create-amp/src/Domain.ts`
4. Template files should include `package.json` (will be modified with project name)
5. Use `_.gitignore` for gitignore template files (underscore prefix to avoid being ignored)

### Template Skip Patterns

Templates use `skip` sets to exclude files during download:
- `Constants.ALWAYS_SKIP_DIRECTORIES` = `[".git", "node_modules"]`
- Add build output directories like `"dist"`, `".next"`, `".tanstack"` as needed

### Package Manager Resolution

CLI prompts for package manager if not specified via `--package-manager` option. Supports: pnpm, npm, yarn, bun.

## Key Technologies

- **Runtime**: Node.js v22+, TypeScript 5.9+
- **Effect-TS**: v3.19.3+ (Effect, @effect/cli, @effect/platform, @effect/schema)
- **CLI**: @effect/cli for commands/prompts, Effect Streams for HTTP downloads
- **Templates**: Next.js 15, Vite, React, Express, Fastify, Apollo GraphQL
- **Build Tool**: tsdown (for CLI package)
- **Testing**: Vitest with Effect Vitest integration (@effect/vitest)
- **Package Management**: pnpm workspaces (v10.22.0+)

## Repository Conventions

- Use workspace protocol for internal dependencies
- No semicolons (Prettier enforced)
- 120 character line width
- Import ordering: builtin, external, parent, sibling (alphabetical within groups)
- Effect naming: PascalCase for errors/services, camelCase for functions
- All async operations use Effect, not Promises

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/edgeandnode)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/edgeandnode)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
