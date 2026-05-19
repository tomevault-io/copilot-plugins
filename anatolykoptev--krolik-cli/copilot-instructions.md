## krolik-cli

> > Internal guide for developing krolik-cli. For **usage** docs, see root `CLAUDE.md`.

# KROLIK CLI — Development Guide

> Internal guide for developing krolik-cli. For **usage** docs, see root `CLAUDE.md`.

**Version:** 0.7.0 | **Node:** >=20 | **Package Manager:** pnpm

## Architecture Principles

| Rule | Enforcement |
|------|-------------|
| SRP | One purpose per file |
| Max lines | 200 |
| Paths | Dynamic detection via `detectSrcPaths()`, never hardcoded |
| Dependencies | Explicit injection |
| Functions | Pure first |
| CLI flags | Extend existing, never add new |

## ⛔ FORBIDDEN: Regex for Code Analysis

**NEVER use regex to analyze or transform code.** Regex cannot reliably distinguish:
- JSX attributes vs object properties vs function arguments
- String literals in different contexts
- Nested structures and edge cases

| ❌ Forbidden | ✅ Required |
|-------------|-------------|
| Regex patterns for parsing | SWC for fast analysis (`lib/@swc`) |
| String matching for context | ts-morph AST for transformations (`lib/@ast`) |
| Line-by-line text replacement | AST-based node replacement |

**Why:** Regex leads to exponential edge cases (11+ patterns for i18n alone). AST gives 100% accuracy via `SyntaxKind`.

**Example — i18n string replacement:**
```typescript
// ❌ WRONG: Regex-based (breaks constantly)
if (/^\w+:\s*["']/.test(line)) { /* object property? maybe... */ }

// ✅ CORRECT: AST-based (always accurate)
if (parent.getKind() === SyntaxKind.PropertyAssignment) { /* definitely object property */ }
```

**See:** [ast-transformer.ts](src/lib/@i18n/ast-transformer.ts) for reference implementation.

<!-- krolik:start -->
<!-- version: 6.1.0 | auto-updated -->

## 🐰 Krolik

### Session Startup

**FIRST:** Call these tools at session start:

1. `krolik_status` `fast: true`
2. `krolik_mem_recent` `limit: 5`
3. `krolik_context` `feature: "..."` or `issue: "123"` (if working on specific feature)

### Context Cache

**FIRST:** Read `.krolik/CONTEXT.xml` — if missing, run `krolik_context -q`

```xml
<context mode="quick|deep|full" generated="ISO-timestamp">
```

| Mode | Sections | Use |
|------|----------|-----|
| `quick` | architecture, git, tree, schema, routes | Fast overview |
| `deep` | imports, types, env, contracts | Heavy analysis |
| `full` | all sections | Complete context |

**Refresh if:** file missing, stale (>1h), or wrong mode

### Tools

| When | Tool | Params |
|------|------|--------|
| **Need progress overview** | `krolik_progress` |  |
| **Session start** | `krolik_status` | `fast: true` |
| **Before feature/issue work** | `krolik_context` | `feature: "..."` or `issue: "123"` |
| **Need library API docs** | `krolik_docs` | `action: "search", query: "..."` |
| **Parse GitHub issue details** | `krolik_issue` | `number: "123"` |
| **Before writing utilities** | `krolik_modules` | `action: "search", query: "..."` |
| **API routes questions** | `krolik_routes` | — |
| **DB schema questions** | `krolik_schema` | — |
| **Code quality audit** | `krolik_audit` | — |
| **Quality issues found** | `krolik_fix` | `dryRun: true` first |
| **Find duplicates/structure** | `krolik_refactor` |  |
| **After code changes** | `krolik_review` | `staged: true` |
| **Get recent memories** | `krolik_mem_recent` | `limit: 5` |
| **Save decision/pattern/bugfix** | `krolik_mem_save` | `type: "decision", title: "..."` |
| **Search memories by query** | `krolik_mem_search` | `query: "authentication"` |
| **Multi-agent orchestration** | `krolik_agent` | `orchestrate: true, task: "..."` |

<!-- krolik:end -->

---

## Project Structure

```
krolik-cli/
├── src/
│   ├── bin/            # CLI entry point (cli.ts)
│   ├── cli/            # Command registration
│   ├── commands/       # Command implementations
│   │   ├── agent/      # Multi-agent orchestration
│   │   ├── audit/      # Code quality audit
│   │   ├── codegen/    # Code generation (templates, generators)
│   │   ├── context/    # AI-context generation (formatters, parsers, repomap)
│   │   ├── docs/       # Library documentation cache (Context7)
│   │   ├── fix/        # Auto-fix (fixers, analyzers, strategies, recommendations)
│   │   ├── init/       # Project initialization
│   │   ├── issue/      # GitHub issue parsing
│   │   ├── memory/     # Persistent memory (SQLite)
│   │   ├── refactor/   # Module refactoring (duplicates, migrations)
│   │   ├── review/     # Code review (git diff analysis)
│   │   ├── routes/     # tRPC routes analysis
│   │   ├── schema/     # Prisma schema analysis
│   │   ├── security/   # Security audit
│   │   ├── setup/      # Plugin installation (diagnostics, installers)
│   │   ├── status/     # Project diagnostics
│   │   └── sync/       # Sync operations
│   ├── config/         # Config loading (cosmiconfig)
│   ├── lib/            # Shared utilities (@-prefixed modules)
│   │   ├── @agents/    # Agent definitions & orchestration
│   │   ├── @ast/       # AST operations (ts-morph pool, SWC parser, fingerprinting)
│   │   ├── @cache/     # SQLite caching layer
│   │   ├── @claude/    # Claude Code integration (sections)
│   │   ├── @core/      # Foundation: fs, shell, logger, time, utils, constants
│   │   ├── @detectors/ # Issue detectors: lint, security, i18n, quality, patterns
│   │   ├── @discovery/ # File discovery: architecture, reusables
│   │   ├── @format/    # Formatters: markdown, xml, json
│   │   ├── @i18n/      # Internationalization (AST transformer, languages)
│   │   ├── @integrations/ # External: Context7
│   │   ├── @prisma/    # Prisma schema parsing
│   │   ├── @ranking/   # PageRank for file importance
│   │   ├── @security/  # Sanitization, secrets detection
│   │   ├── @storage/   # Memory & docs storage (SQLite)
│   │   ├── @tokens/    # Token counting (gpt-tokenizer)
│   │   └── @vcs/       # Version Control (Git, GitHub)
│   ├── mcp/            # MCP server
│   │   ├── tools/      # MCP tool implementations (14 tools)
│   │   ├── handlers.ts
│   │   ├── resources.ts
│   │   └── server.ts
│   └── types/          # TypeScript types
├── tests/
│   ├── unit/           # Unit tests (mirror src/)
│   ├── fixtures/       # Test data
│   ├── helpers/        # Test utilities
│   └── scripts/        # Test scripts
└── .changeset/         # Changesets for releases
```

## Coding Standards

```typescript
// Types: explicit, never any
function analyze(files: string[]): AnalysisResult { }

// Exports: named only
export { runStatus } from './status';

// Imports order: node: → external → @/ → ./
import * as fs from 'node:fs';
import { Command } from 'commander';
import { getConfig } from '@/config';
import { format } from './output';

// Errors: Result type
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };
```

## Critical: AST System

**ALWAYS use the unified pool from `lib/@ast` to prevent memory leaks.**

```typescript
// RECOMMENDED: Auto-cleanup callback pattern
import { withSourceFile } from '@/lib/@ast';

const count = withSourceFile(content, 'temp.ts', (sourceFile) => {
  return sourceFile.getFunctions().length;
});

// ADVANCED: Manual project management
import { getProject, releaseProject } from '@/lib/@ast';

const project = getProject({ tsConfigPath: './tsconfig.json' });
try {
  // ... use project
} finally {
  releaseProject(project);
}
```

**Key points:**
- Pool reuses Project instances to avoid memory leaks
- `withSourceFile()` — single-file operations
- `getProject()` + `releaseProject()` — multi-file operations
- Never use deprecated `createProject()` directly

## Critical: Discovery & Detection

**NEVER hardcode paths or patterns.** Use discovery modules:

```typescript
// Architecture discovery
import { detectArchitecture, detectMonorepoPackages } from '@/lib/@discovery';
import { detectSrcPaths } from '@/config/detect';

const packages = await detectMonorepoPackages(projectRoot);
const srcPaths = detectSrcPaths(projectRoot, 'src');

// Issue detection (lint, security, i18n, quality)
import { detectLintIssue, detectSecurityIssue } from '@/lib/@detectors';

const issues = detectLintIssue(content, filePath, lineNumber);
```

## Principle: Zero False Positives > High Recall

Google approach — better to miss a real issue than show a false one.

## SWC Fast Parsing

For performance-critical parsing (10-50x faster than ts-morph):

```typescript
import { parseFile } from '@/lib/@ast/swc/parser';

// parseFile returns AST with offset normalization for span handling
const { ast, lineOffsets, baseOffset } = parseFile(filePath, content);

// IMPORTANT: SWC spans accumulate across calls, use baseOffset to normalize:
const normalizedStart = span.start - baseOffset - 1; // -1 for 1-based to 0-based
```

For function extraction use the refactor analyzer:

```typescript
import { extractFunctionsSwc } from '@/commands/refactor/analyzers/core/swc-parser';

const functions = extractFunctionsSwc(filePath, content);
// Returns: SwcFunctionInfo[] with name, line, bodyHash, fingerprint, etc.
```

## Refactor Command Architecture

**Path detection is ALWAYS dynamic.** The refactor command analyzes ALL source directories, not just `lib/`.

### Key Principles

| Rule | Implementation |
|------|----------------|
| Dynamic paths | Use `detectSrcPaths()` from config |
| No hardcoded dirs | Pattern-based detection |
| No new CLI flags | Extend existing infrastructure |
| Multi-path analysis | Parallel scanning + merge |

### Source Path Detection

```typescript
// config/detect.ts
export function detectSrcPaths(projectRoot: string, pkgPath: string): string[]
// Scans pkgPath for directories containing .ts/.tsx files
// Returns: ['apps/web/lib', 'apps/web/components', 'apps/web/app', ...]
```

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    refactor command                          │
├─────────────────────────────────────────────────────────────┤
│  1. resolvePaths()                                          │
│     ├─ Monorepo? → detectMonorepoPackages() → srcPaths     │
│     └─ Single?   → detectSrcPaths(projectRoot, 'src')       │
│                                                              │
│  2. runRefactor()                                            │
│     ├─ Parallel analysis on each path                        │
│     └─ mergeDuplicates() + mergeStructures()                │
│                                                              │
│  3. Output: path shows all analyzed dirs                     │
│     p="src/lib, src/commands, src/mcp, src/config"          │
└─────────────────────────────────────────────────────────────┘
```

### File Locations

| File | Purpose |
|------|---------|
| [src/config/detect.ts](src/config/detect.ts) | `detectSrcPaths()`, `MonorepoPackage.srcPaths` |
| [src/commands/refactor/paths/resolver.ts](src/commands/refactor/paths/resolver.ts) | Path resolution for mono/single |
| [src/commands/refactor/runner/analysis.ts](src/commands/refactor/runner/analysis.ts) | Parallel analysis + merge utils |

### Examples

**Monorepo (piternow):**
```
📦 Monorepo detected. Analyzing: web [__tests__, app, components, lib]
   Available packages: mobile, web, api, ui
```

**Single project (krolik-cli):**
```
📁 Single project. Analyzing: [bin, cli, commands, config, lib, mcp, types]
```

## Adding New Command

1. Create folder in `src/commands/<name>/`
2. Structure:
   ```
   commands/<name>/
   ├── index.ts      # Command registration
   ├── run.ts        # Main logic
   ├── types.ts      # Types
   └── CLAUDE.md     # (optional) Command-specific docs
   ```
3. Register in `src/commands/index.ts`
4. Add MCP tool in `src/mcp/tools/<name>/`

## Adding MCP Tool

1. Create folder in `src/mcp/tools/<name>/`
2. Structure:
   ```
   tools/<name>/
   ├── index.ts      # Tool definition
   ├── handler.ts    # Handler logic
   └── schema.ts     # Zod schema for params
   ```
3. Register in `src/mcp/tools/index.ts`

## Testing

```bash
pnpm test              # Watch mode
pnpm test:run          # Single run
pnpm test:coverage     # With coverage
```

- Tests in `tests/unit/` mirror `src/` structure
- Fixtures in `tests/fixtures/`
- Name pattern: `*.test.ts`

## Development

```bash
pnpm dev              # Watch mode (tsx)
pnpm build            # Build to dist/
pnpm typecheck        # Type check
pnpm lint             # Lint (biome)
pnpm check:fix        # Fix lint + format

# Test CLI locally
./dist/bin/cli.js status --fast
./dist/bin/cli.js context --quick
```

## ⛔ Release: ONLY via Changesets

**NEVER manually change version in package.json!**

| ❌ Forbidden | ✅ Required |
|-------------|-------------|
| `npm version patch/minor/major` | `pnpm changeset` |
| Edit `package.json` version | Create `.changeset/*.md` file |
| Manual `npm publish` | Push to GitHub → Actions publish |

**Release workflow:**

```bash
# 1. Create changeset (describes changes)
pnpm changeset

# 2. Commit and push
git add .changeset/*.md
git commit -m "chore: add changeset for <feature>"
git push

# 3. GitHub Actions automatically:
#    - Runs changeset version (bumps package.json)
#    - Publishes to npm via Trusted Publisher
#    - Creates GitHub release
```

**Why:** Trusted Publisher uses OIDC (no npm tokens), ensures provenance, publishes to both npm and GitHub releases atomically.

## Key Locations

| What | Path |
|------|------|
| CLI Entry | [src/bin/cli.ts](src/bin/cli.ts) |
| MCP Server | [src/mcp/server.ts](src/mcp/server.ts) |
| MCP Tools | [src/mcp/tools/](src/mcp/tools/) |
| AST (ts-morph + SWC) | [src/lib/@ast/](src/lib/@ast/) |
| Issue Detectors | [src/lib/@detectors/](src/lib/@detectors/) |
| Agent System | [src/lib/@agents/](src/lib/@agents/) |
| Fix Command Docs | [src/commands/fix/CLAUDE.md](src/commands/fix/CLAUDE.md) |
| Lib Internal Docs | [src/lib/CLAUDE.md](src/lib/CLAUDE.md) |

## Dependencies

**Core:**
- `commander` — CLI framework
- `cosmiconfig` — Config loading
- `ts-morph` — AST analysis (pooled)
- `better-sqlite3` — Memory/cache storage
- `gpt-tokenizer` — Token counting
- `zod` — Schema validation
- `chalk` — Terminal styling
- `glob` — File matching
- `prettier` — Code formatting
- `@swc/core` — Fast parsing (10-50x faster than ts-morph)

**Dev:**

- `@biomejs/biome` — Lint + format
- `vitest` — Testing
- `tsup` — Build
- `@changesets/cli` — Release management
- `husky` + `lint-staged` — Pre-commit hooks

---
> Source: [anatolykoptev/krolik-cli](https://github.com/anatolykoptev/krolik-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
