## agent-claude-code

> [TokenTop](https://github.com/tokentopapp/tokentop) is a terminal-based dashboard for monitoring

# AGENTS.md — @tokentop/agent-claude-code

## What is TokenTop?

[TokenTop](https://github.com/tokentopapp/tokentop) is a terminal-based dashboard for monitoring
AI token usage and costs across providers and coding agents. It uses a plugin architecture
(`@tokentop/plugin-sdk`) with four plugin types: **provider** (API cost fetching), **agent**
(session parsing), **theme** (TUI colors), and **notification** (alerts).

This package is an **agent plugin**. Agent plugins parse local session files written by coding
agents (Claude Code, Cursor, etc.) to extract per-turn token usage, then feed normalized
`SessionUsageData` rows back to the TokenTop core for display. This plugin specifically parses
Claude Code (Anthropic CLI) JSONL files from `~/.claude/projects/`.

## Build & Run

```bash
bun install                  # Install dependencies
bun run build                # Full build (types + JS bundle)
bun run build:types          # tsc --emitDeclarationOnly
bun run build:js             # bun build → dist/
bun run typecheck            # tsc --noEmit (strict)
bun test                     # Run all tests (bun test runner)
bun test tests/parser.test.ts  # Run a single test file
bun test --watch             # Watch mode
```

CI runs `bun run build` then `bun run typecheck`. Both must pass.

## Project Structure

```
src/
├── index.ts      # Plugin entry — createAgentPlugin(), exports
├── types.ts      # Claude Code JSONL format types
├── parser.ts     # Session file parsing, caching, aggregation
├── paths.ts      # Path constants (~/.claude), directory helpers
├── cache.ts      # Cache state, TTL, eviction
├── utils.ts      # JSONL reader, path decoding
└── watcher.ts    # FSWatcher for session + activity tracking
tests/
├── parser.test.ts
├── cache.test.ts
├── utils.test.ts
└── watcher.test.ts
```

## TypeScript Configuration

- **Strict mode**: `strict: true` — all strict checks enabled
- **No unused code**: `noUnusedLocals`, `noUnusedParameters` both `true`
- **No fallthrough**: `noFallthroughCasesInSwitch: true`
- **Target**: ESNext, Module: ESNext, ModuleResolution: bundler
- **Types**: `bun-types` (not `@types/node`)
- **Declaration**: Emits `.d.ts` + declaration maps + source maps

## Code Style

### Imports

- **Use `.ts` extensions** in all relative imports: `import { foo } from './bar.ts'`
- **Type-only imports** use the `type` keyword:
  ```typescript
  import type { SessionUsageData } from '@tokentop/plugin-sdk';
  import { createAgentPlugin, type AgentFetchContext } from '@tokentop/plugin-sdk';
  ```
- **Node.js modules** via namespace imports: `import * as fs from 'fs'`, `import * as path from 'path'`
- **Order**: External packages → relative imports (no blank line separator used)

### Module Format

- ESM only (`"type": "module"` in package.json)
- Named exports for everything except the main plugin (default export)
- Re-export public API items explicitly from `index.ts`

### Naming Conventions

- **Constants**: `UPPER_SNAKE_CASE` — `CACHE_TTL_MS`, `RECONCILIATION_INTERVAL_MS`
- **Functions**: `camelCase` — `parseSessionsFromProjects`, `readJsonlFile`
- **Interfaces**: `PascalCase` — `ClaudeCodeAssistantEntry`, `SessionWatcherState`
- **Type predicates**: `is` prefix — `isTokenBearingAssistant(entry): entry is ClaudeCodeAssistantEntry`
- **Unused required params**: Underscore prefix — `_ctx: PluginContext`
- **File names**: `kebab-case.ts`

### Types

- **Interfaces** for object shapes, not type aliases
- **Explicit return types** on all exported functions
- **Type predicates** for runtime validation guards (narrowing `unknown` → typed)
- **`Partial<T>`** for candidate validation instead of `as any`
- Never use `as any`, `@ts-ignore`, or `@ts-expect-error`
- Validate unknown data at boundaries with type guard functions

### Functions

- **Functional style** — no classes. State held in module-level objects/Maps
- **Pure functions** where possible; side effects isolated to watcher/cache modules
- **Early returns** for guard clauses
- **Async/await** throughout, no raw Promise chains

### Error Handling

- **Empty catch blocks are intentional** for graceful degradation (filesystem ops that may fail)
- Pattern: `try { await fs.access(path); } catch { return []; }`
- Never throw from filesystem operations — return empty/default values
- Use `Number.isFinite()` for numeric validation, not `isNaN()`
- Validate at data boundaries, trust within module

### Formatting

- No explicit formatter config (Prettier/ESLint not configured)
- 2-space indentation (observed convention)
- Single quotes for strings
- Trailing commas in multiline structures
- Semicolons always
- Opening brace on same line

## Plugin SDK Contract

The plugin SDK (`@tokentop/plugin-sdk`) defines the interface contract between plugins and
the TokenTop core (`~/development/tokentop/ttop`). The SDK repo lives at
`~/development/tokentop/plugin-sdk`. This plugin is a peer dependency consumer — it declares
`@tokentop/plugin-sdk` as a `peerDependency`, not a bundled dep.

This plugin implements the `AgentPlugin` interface via the `createAgentPlugin()` factory:

```typescript
const plugin = createAgentPlugin({
  id: 'claude-code',
  type: 'agent',
  agent: { name: 'Claude Code', command: 'claude', configPath, sessionPath },
  capabilities: { sessionParsing: true, realTimeTracking: true, ... },
  isInstalled(ctx) { ... },
  parseSessions(options, ctx) { ... },
  startActivityWatch(ctx, callback) { ... },
  stopActivityWatch(ctx) { ... },
});
export default plugin;
```

### AgentPlugin interface (required methods)

| Method | Signature | Purpose |
|--------|-----------|---------|
| `isInstalled` | `(ctx: PluginContext) → Promise<boolean>` | Check if Claude Code exists on this machine |
| `parseSessions` | `(options: SessionParseOptions, ctx: AgentFetchContext) → Promise<SessionUsageData[]>` | Parse JSONL files into normalized usage rows |
| `startActivityWatch` | `(ctx: PluginContext, callback: ActivityCallback) → void` | Begin real-time file watching, emit deltas |
| `stopActivityWatch` | `(ctx: PluginContext) → void` | Tear down watchers |

### Key SDK types

| Type | Shape | Used for |
|------|-------|----------|
| `SessionUsageData` | `{ sessionId, providerId, modelId, tokens: { input, output, cacheRead?, cacheWrite? }, timestamp, sessionUpdatedAt?, projectPath?, sessionName? }` | Normalized per-turn usage row returned from `parseSessions` |
| `ActivityUpdate` | `{ sessionId, messageId, tokens: { input, output, cacheRead?, cacheWrite? }, timestamp }` | Real-time delta emitted via `ActivityCallback` |
| `SessionParseOptions` | `{ sessionId?, limit?, since?, timePeriod? }` | Filters passed by core to `parseSessions` |
| `AgentFetchContext` | `{ http, logger, config, signal }` | Context bag — `ctx.logger` for debug logging |
| `PluginContext` | `{ logger, storage, config, signal }` | Context for lifecycle methods |

### SDK subpath imports

| Import path | Use |
|-------------|-----|
| `@tokentop/plugin-sdk` | Everything (types + helpers) |
| `@tokentop/plugin-sdk/types` | Type definitions only |
| `@tokentop/plugin-sdk/testing` | `createTestContext()` for tests |

## Architecture Notes

- **JSONL parsing**: Claude Code stores sessions as append-only JSONL in `~/.claude/projects/<slug>/<session-id>.jsonl`
- **Two-layer caching**: `sessionCache` (TTL-based full result cache) + `sessionAggregateCache` (per-session parsed rows, LRU eviction at 10K entries)
- **Dirty tracking**: FSWatcher marks changed files; only dirty/new files get re-stat'd and re-parsed
- **Reconciliation**: Full stat sweep forced every 10 minutes via interval timer
- **Activity watching**: Delta reads (file offset tracking) for real-time streaming — reads only new bytes appended since last check
- **Deduplication**: Uses `message.id` as dedup key within each session

## Commit Conventions

Conventional Commits enforced by CI on both PR titles and commit messages:

```
feat(parser): add support for cache_creation breakdown
fix(watcher): handle race condition in delta reads
chore(deps): update dependencies
refactor: simplify session metadata indexing
```

Valid prefixes: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`
Optional scope in parentheses. Breaking changes use `!` suffix before colon.

## Release Process

- semantic-release via GitHub Actions (currently manual `workflow_dispatch`)
- Publishes to npm as `@tokentop/agent-claude-code` with public access + provenance
- Runs `bun run clean && bun run build` before publish (`prepublishOnly`)
- Branches: `main` only

## Testing

- Test runner: `bun test` (Bun's built-in test runner)
- Test files: `*.test.ts` in `tests/` directory (excluded from tsconfig compilation, picked up by bun test)
- Place test files in `tests/`: `tests/parser.test.ts`
- Imports use relative paths back to source: `import { foo } from '../src/foo.ts'`

---
> Source: [tokentopapp/agent-claude-code](https://github.com/tokentopapp/agent-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
