## ls-apis

> ├── package.json           # root (workspaces)

# ls-apis

## Project Structure

```
ls-apis/
├── package.json           # root (workspaces)
├── qa-output/             # QA reports (gitignored)
├── packages/
│   ├── aggregator/        # fetches, normalizes, deduplicates
│   │   ├── src/
│   │   │   ├── aggregate.ts       # main orchestration
│   │   │   ├── normalize.ts       # entry & category normalization
│   │   │   ├── paths.ts           # path utilities
│   │   │   ├── qa/                # QA validation
│   │   │   │   ├── index.ts       # QA orchestrator
│   │   │   │   ├── validations.ts # pure validation functions
│   │   │   │   └── tests/         # QA-specific tests
│   │   │   ├── sources/           # pluggable fetchers (*.fetcher.ts)
│   │   │   │   ├── index.ts       # fetcher auto-loader
│   │   │   │   └── tests/         # fetcher-specific tests
│   │   │   ├── tests/             # aggregator tests
│   │   │   └── types.ts           # ApiEntry, SourceFetcher interfaces
│   │   └── vitest.config.ts
│   ├── cli/               # CLI for searching APIs
│   │   ├── data/
│   │   │   └── apis.json          # bundled API data (published with package)
│   │   ├── src/
│   │   │   ├── index.ts           # CLI entry point
│   │   │   ├── paths.ts           # workspace root resolution
│   │   │   ├── colors.ts          # terminal color support
│   │   │   ├── qa.ts              # QA command handler
│   │   │   ├── categories.ts      # categories command
│   │   │   ├── providers.ts       # providers command
│   │   │   └── formatter.ts       # output formatter
│   │   └── tests/
│   │       ├── paths.test.ts      # path resolution tests
│   │       ├── qa.test.ts         # QA wrapper tests
│   │       ├── cli.test.ts        # CLI integration tests
│   │       ├── categories.test.ts # categories command tests
│   │       ├── providers.test.ts  # providers command tests
│   │       └── config.test.ts     # config tests
│   ├── shared/             # Shared types, config, search, paths (consumed by all packages)
│   │   ├── src/
│   │       ├── index.ts
│   │       ├── types.ts           # ApiEntry, DataFile, Provider, SearchOptions
│   │       ├── config.ts          # Config loading/display (moved from CLI)
│   │       ├── search.ts          # Search logic (moved from CLI)
│   │       └── paths.ts           # Workspace root resolution
│   └── mcp-server/         # MCP server for AI-friendly API queries (stdio transport)
│       └── src/
│           ├── index.ts           # Entry point
│           ├── server.ts          # MCP server with tools & resources
│           └── data.ts            # Data loading (apis.json)
└── AGENTS.md              # instructions for AI agents
```

## Commands

```bash
# Install deps
npm install

# Run aggregator (fetch all sources → dedupe → packages/cli/data/apis.json)
npm run aggregate

# Run QA checks on aggregated data (output to qa-output/issues.json)
npm run qa

# Run tests with coverage (both aggregator + cli)
npm test

# Run specific package tests
npm run test:aggregator
npm run test:cli

# Typecheck all workspaces
npm run typecheck

# MCP server (stdio transport for AI clients)
npm run mcp

# Lint & format
npm run lint
npm run lint:fix
npm run format
npm run format:fix

# CLI search (via npm script)
npm run ls-apis -- -q <query>
npm run ls-apis -- -c <category>
npm run ls-apis -- -a <auth>

# Build all packages (shared → CLI)
npm run build

# Build CLI only (requires shared to be built first)
npm run build --workspace=@ls-apis/cli

# CLI search (from compiled output, workspace root)
node packages/cli/dist/index.js -q <query>

# CLI search (globally, after npm link)
npm link --workspace=@ls-apis/cli
ls-apis -q <query>
```

## CLI Build Notes

- `@ls-apis/cli` publishes `dist/`.
- CLI package build uses `tsc` then `tsc-esm-fix --target dist`.
- `tsc-esm-fix` is required because Node ESM runtime requires explicit `.js` extensions in relative imports.
- CLI depends on `@ls-apis/shared` — shared must be built first (`npm run build` at root handles this).

## Shared Package Notes

- `@ls-apis/shared` uses `.js` extensions on all relative imports (required for Node.js ESM).
- Source `.ts` files import with `.js` extensions (e.g., `from './paths.js'`), which TypeScript resolves to `.ts` during compilation.
- `npm run build` at root builds shared first, then CLI.

## Architecture

- **Fetchers**: `*.fetcher.ts` files in `packages/aggregator/src/sources/`
- **Naming convention**: must end with `.fetcher.ts`
- **Interface**: implements `SourceFetcher` (name + sourceUrl + fetchApis())
- **Auto-loading**: via `loadAllFetchers()` in `sources/index.ts`
- **CLI colors**: `src/colors.ts` handles terminal coloring with chalk, respects `NO_COLOR` env and `--no-color` flag

## MCP Server Notes

- **MCP client config**: use `npx tsx packages/mcp-server/src/index.ts` for all platforms. VS Code will ask for permission once on first run — this is normal for project-local MCP servers (as opposed to published npm packages which are pre-trusted). Approving once persists the decision.
- **Auto-build**: `packages/mcp-server/index.js` is a small JS shim that builds the server (`tsc && tsc-esm-fix`) if `dist/` doesn't exist, then delegates to `dist/index.js`. This ensures MCP client configs work on fresh clones without a manual build step.
- **SDK imports use `.js`**: `@modelcontextprotocol/sdk@0.5.0` has a wildcard exports map (`"./*": "./dist/*"`) that TypeScript's `bundler` resolution can't resolve without the extension ([#218](https://github.com/modelcontextprotocol/typescript-sdk/issues/218), [#258](https://github.com/modelcontextprotocol/typescript-sdk/issues/258)). The MCP server source uses `.js` extensions on these SDK imports.

## Data Schema

```typescript
interface DataFile {
  timestamp: string; // ISO 8601 UTC timestamp of processing
  providers: Provider[]; // Data source providers
  apis: ApiEntry[]; // Aggregated API entries
}

interface Provider {
  name: string; // Provider identifier (e.g., 'apis-guru')
  url: string; // Data source URL
}

interface ApiEntry {
  name: string;
  description?: string;
  link: string;
  auth?: string;
  cors?: string;
  categories: string[];
  openapiSpec?: string;
  sources: string[];
}
```

## Adding a New Source

1. Create `packages/aggregator/src/sources/<name>.fetcher.ts`
2. Implement `SourceFetcher` interface:

   ```typescript
   import type { SourceFetcher, ApiEntry } from '../types';

   export const mysourceFetcher: SourceFetcher = {
     name: 'mysource',
     sourceUrl: 'https://example.com/api',
     fetchApis: async (): Promise<ApiEntry[]> => {
       // Fetch and normalize APIs from your source
       return [
         /* ApiEntry items */
       ];
     },
   };
   ```

3. Tests go in `packages/aggregator/src/sources/tests/<name>.test.ts`
4. Run `npm run aggregate` to fetch and update `packages/cli/data/apis.json`

## CLI Options

| Flag         | Alias | Description                              |
| ------------ | ----- | ---------------------------------------- |
| `--query`    | `-q`  | Search query (filters name, description) |
| `--category` | `-c`  | Filter by category                       |
| `--auth`     | `-a`  | Filter by auth (apiKey, OAuth, no)       |
| `--limit`    | `-l`  | Max results (from config or default: 20) |
| `--output`   | `-o`  | Output format: text or json              |
| `--sort`     | `-s`  | Sort results: name, category, auth       |
| `--no-color` |       | Disable colored output                   |
| `--help`     | `-h`  | Show help                                |
| `--version`  | `-V`  | Show version                             |

## CLI Commands

| Command      | Description                         |
| ------------ | ----------------------------------- |
| `categories` | List all API categories with counts |
| `providers`  | List all data providers             |
| `config`     | Show config settings and file path  |
| `qa`         | Run QA checks (terminal summary)    |

### QA Options

| Flag     | Alias | Description                      |
| -------- | ----- | -------------------------------- |
| `--file` | `-f`  | Output file path for JSON report |

### Categories Options

| Flag       | Alias | Description                         |
| ---------- | ----- | ----------------------------------- |
| `--sort`   | `-s`  | Sort by: name (default), count      |
| `--output` | `-o`  | Output format: text (default), json |

### Providers Options

| Flag       | Alias | Description                         |
| ---------- | ----- | ----------------------------------- |
| `--sort`   | `-s`  | Sort by: name (default), count      |
| `--output` | `-o`  | Output format: text (default), json |

## Config File

`~/.ls-apis` (JSON) is created automatically on first CLI run. Edit it to customize defaults. CLI flags always override.

**Location**: `~/.ls-apis` (user's home directory)

```json
{
  "limit": 10,
  "descriptionMaxLength": 250,
  "colors": true
}
```

| Key                    | Default | Description                 |
| ---------------------- | ------- | --------------------------- |
| `limit`                | 20      | Default max results         |
| `descriptionMaxLength` | 250     | Max chars before truncation |
| `colors`               | true    | Enable terminal colors      |

Uses only Node.js stdlib (`os.homedir()`, `fs/promises`). Missing or invalid file falls back to built-in defaults.

## Relevant Files

- `packages/cli/src/index.ts`: CLI entry point with `categories`, `providers`, `config`, `qa` commands
- `packages/cli/src/qa.ts`: `ls-apis qa` handler — shells out to aggregator QA via `execSync`
- `packages/cli/src/categories.ts`: Categories command handler
- `packages/cli/src/providers.ts`: Providers command handler
- `packages/cli/src/paths.ts`: Workspace root resolution (`projectRoot`)
- `packages/cli/tests/paths.test.ts`: Tests for path resolution
- `packages/cli/tests/qa.test.ts`: Tests for CLI qa wrapper
- `packages/aggregator/src/aggregate.ts`: Aggregation orchestration, deduplication
- `packages/aggregator/src/normalize.ts`: Entry & category normalization
- `packages/aggregator/src/qa/index.ts`: QA orchestrator (reads apis.json, runs validations, outputs grouped JSON)
- `packages/aggregator/src/qa/validations.ts`: Pure validation functions
- `packages/aggregator/src/paths.ts`: Path utilities for Windows ESM compatibility
- `packages/aggregator/src/sources/apis-guru.fetcher.ts`, `.fetcher.ts`: Fetcher implementations
- `packages/aggregator/src/sources/index.ts`: Fetcher auto-loader
- `packages/shared/src/types.ts`: Unified shared types (ApiEntry, DataFile, Provider, SearchOptions)
- `packages/shared/src/config.ts`: Config loading from `~/.ls-apis`
- `packages/shared/src/search.ts`: Search/filter/sort logic
- `packages/shared/src/paths.ts`: Workspace root resolution
- `packages/mcp-server/src/index.ts`: MCP server entry point
- `packages/mcp-server/src/server.ts`: MCP server with tools & resources
- `packages/mcp-server/src/data.ts`: Data loading for MCP server

---
> Source: [koalyptus/ls-apis](https://github.com/koalyptus/ls-apis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
