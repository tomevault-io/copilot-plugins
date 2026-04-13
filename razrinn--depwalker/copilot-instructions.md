## depwalker

> > **For AI Coding Agents:** Essential context for working with the DepWalker codebase.

# DepWalker - Agent Documentation

> **For AI Coding Agents:** Essential context for working with the DepWalker codebase.

## Project Overview

DepWalker is a TypeScript dependency analysis CLI tool that tracks the impact of code changes across a codebase. It analyzes Git changes and shows which functions are affected, along with their dependency chains.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              CLI Entry                                  │
│                         (src/index.ts)                                  │
│                     Parses args, orchestrates flow                      │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
           ┌────────────────────────┼────────────────────────┐
           ▼                        ▼                        ▼
  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
  │   Git Module    │    │ Analyzer Module │    │ Formatter Module│
  │   (src/git.ts)  │    │(src/analyzer.ts)│    │(src/formatter.ts│
  │                 │    │                 │    │                 │
  │ • git diff -U0  │    │ • Create TS     │    │ • Route to      │
  │ • Parse hunks   │    │   Program       │    │   plugins       │
  │ • Extract line  │    │ • Build Call    │    │                 │
  │   ranges        │    │   Graph (AST)   │    │                 │
  └────────┬────────┘    │ • Find changed  │    └────────┬────────┘
           │             │   functions     │             │
           │             └────────┬────────┘             │
           │                      │                      │
           └──────────────────────┼──────────────────────┘
                                  ▼
                    ┌─────────────────────────┐
                    │     AnalysisResult      │
                    │  ┌─────────────────┐    │
                    │  │ changedFiles    │    │
                    │  │ changedFunctions│    │
                    │  │ callGraph       │    │
                    │  └─────────────────┘    │
                    └─────────────────────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    ▼                           ▼
         ┌─────────────────────┐    ┌─────────────────────┐
         │  Markdown Plugin    │    │    HTML Plugin      │
         │(format-markdown/)   │    │   (format-html/)    │
         │                     │    │                     │
         │ • Compact tables    │    │ • Single-page view  │
         │ • Inline summary    │    │ • Collapsible trees │
         │ • Test targets list │    │ • Test targets tbl  │
         └─────────────────────┘    └─────────────────────┘
```

### Data Flow

1. **Input**: Uncommitted Git changes in a TypeScript project
2. **Git Parsing**: `getGitDiff()` → `parseGitDiff()` → `Map<filepath, lineNumbers[]>`
3. **AST Analysis**: TS Compiler API builds `Program` → traverses AST → `CallGraph`
4. **Impact Tracing**: For each changed function, walk the call graph to find dependents
5. **Output**: Plugin generates formatted report (Markdown/HTML)

### Key Features

- **Function Impact Analysis**: Traces function call chains from changed functions
- **Git Integration**: Automatically detects uncommitted changes via `git diff`
- **TypeScript Compiler API**: Uses native TS compiler for accurate AST analysis
- **Plugin-Based Output**: Extensible format system with Markdown and HTML plugins built-in
- **Circular Dependency Handling**: Detects and breaks cycles in the call graph
- **React Pattern Support**: Handles memo, lazy, JSX, and HOC patterns

## Analysis Capabilities

### Supported Function Types

The analyzer (`src/analyzer.ts`) recognizes these function declarations:

| Type                      | AST Node                                        | Example                     |
| ------------------------- | ----------------------------------------------- | --------------------------- |
| Function Declaration      | `FunctionDeclaration`                           | `function foo() {}`         |
| Arrow Function (variable) | `VariableDeclaration` with `ArrowFunction`      | `const foo = () => {}`      |
| Function Expression       | `VariableDeclaration` with `FunctionExpression` | `const foo = function() {}` |
| Method Declaration        | `MethodDeclaration`                             | `class X { foo() {} }`      |
| Object Property Method    | `PropertyAssignment` with function              | `{ setTab: (id) => {} }`    |
| Shorthand Method          | `MethodDeclaration` in object                   | `{ setTab(id) {} }`         |

### Supported Call Patterns

The analyzer tracks these call patterns and maps them to their declarations:

| Pattern                   | Detection Method                             | Notes                           |
| ------------------------- | -------------------------------------------- | ------------------------------- |
| Direct function calls     | `CallExpression` with identifier             | Standard function calls         |
| JSX Elements              | `JsxOpeningElement`, `JsxSelfClosingElement` | React component usage           |
| JSX Expression Containers | `JsxExpression` with identifier              | `{variable}` rendering          |
| Wrapped functions         | Unwraps `React.memo`, `memo()` calls         | Tracks the underlying component |

### Import Resolution

**Supported:**

- Static ES imports: `import { foo } from './foo'`
- Dynamic lazy imports: `lazy(() => import('./Component'))`
- Re-exports through aliases

**Not Supported:**

- Template literal dynamic imports: `` import(`./${name}`) ``
- Require statements: `require('./foo')` (common JS)

### Special React Patterns

The analyzer has specific handling for:

1. **React.memo() / memo()**

   - Detects wrapped components
   - Links memoized component to original
   - File: `src/analyzer.ts` (lines 227-237)

2. **lazy() / React.lazy()**

   - Resolves lazy-loaded module paths
   - Creates caller relationships to all exports from target module
   - File: `src/analyzer.ts` (lines 36-85, 487-537)

3. **Component Aliases**

   - Tracks `const Widget = SomeComponent` patterns
   - Resolves through the alias chain
   - File: `src/analyzer.ts` (lines 239-307)

4. **Callbacks (useMemo, useEffect, etc.)**
   - Traces callbacks passed to hooks back to parent component
   - Uses `callbackParents` Map to track ownership
   - File: `src/analyzer.ts` (lines 145, 461-476)

### State Management Patterns

**Zustand Stores:**

```typescript
const useStore = create<Store>((set, get) => ({
  // These are now tracked as separate functions:
  setTab: (id, data) => {
    // ← Detected as "setTab"
    set({ tab: data });
  },
  removeTab: (id) => {
    // ← Detected as "removeTab"
    set({ tabs: [] });
  },
}));
```

Each property with a function value is tracked separately. This allows granular impact analysis when only one store action is modified.

## Technology Stack

| Component          | Technology                          |
| ------------------ | ----------------------------------- |
| Language           | TypeScript 5.9+                     |
| Runtime            | Node.js 18+                         |
| Package Manager    | pnpm 10.11.0                        |
| Build Tool         | TypeScript Compiler (tsc) + esbuild |
| CLI Framework      | Commander.js                        |
| Release Management | Changesets                          |

## Project Structure

```
depwalker/
├── src/                      # Source code
│   ├── index.ts             # CLI entry point (Commander.js setup)
│   ├── git.ts               # Git diff parsing
│   ├── analyzer.ts          # Core analysis logic (AST analysis, call graph)
│   ├── formatter.ts         # Formatter entry point (plugin orchestration)
│   ├── spinner.ts           # CLI spinner/progress indicator
│   ├── constants.ts         # Shared constants (thresholds, weights)
│   ├── types.ts             # TypeScript interfaces
│   └── plugin/              # Format plugin system
│       ├── types.ts         # Plugin interface definitions
│       ├── registry.ts      # Plugin registry
│       ├── index.ts         # Plugin exports
│       ├── shared/          # Shared utilities for plugins
│       │   ├── utils.ts     # Impact calculation, stats, helpers
│       │   └── tree-builder.ts  # Tree building and test target collection
│       ├── format-markdown/ # Markdown format plugin
│       │   └── index.ts
│       └── format-html/     # HTML format plugin (single-page static)
│           ├── index.ts     # Plugin class
│           ├── types.ts     # Type imports
│           ├── styles.ts    # Minimal clean CSS
│           ├── templates.ts # HTML rendering (tree, nodes, entry points)
│           └── scripts.ts   # Minimal JS (tree collapse/expand)
├── scripts/                  # Build scripts
│   └── build.js             # esbuild bundler with version injection
├── dist/                     # Compiled & bundled output
├── .changeset/               # Changeset files for versioning
├── .github/workflows/        # CI/CD pipelines
├── package.json
├── tsconfig.json
├── README.md                # User documentation
├── AGENTS.md                # This file - agent documentation
├── CONTRIBUTING.md          # Contribution guidelines
└── RELEASE.md               # Release process guide
```

## Build Process

Two-stage build:

1. **TypeScript Compilation**: `tsc` compiles `.ts` files to `.js` in `dist/`
2. **Bundling**: `esbuild` bundles into single `dist/index.js`

```bash
pnpm build    # Full build
pnpm dev      # Watch mode
```

## Code Architecture

### Module Breakdown

#### 1. `src/index.ts` - CLI Entry Point

- Sets up Commander.js CLI
- Orchestrates the analysis pipeline
- Handles progress indicators (spinner)

**Options:**

- `--depth <number>`: Maximum analysis depth
- `--tsconfig <path>`: Custom tsconfig path
- `--output <file>`: Save report to file (default: temp file for HTML)
- `--format <format>`: Output format (`markdown`, `html`)
- `--no-open`: Disable auto-opening browser for HTML format

#### 2. `src/git.ts` - Git Integration

- `getGitDiff()`: Runs `git diff -U0 HEAD`
- `parseGitDiff()`: Extracts changed line numbers per file

#### 3. `src/analyzer.ts` - Core Analysis

- `createTsProgram()`: Creates TS compiler program
- `buildCallGraph()`: Builds function call graph using TS AST
- `findChangedFunctions()`: Identifies functions in changed line ranges

**Key Data Structure:**

```typescript
type CallGraph = Map<string, FunctionInfo>;
// "filepath:funcName" -> {
//   callers: CallSite[],
//   definition: { startLine: number, endLine: number },
//   lazyImports?: LazyImport[]
// }
```

**CallGraph Construction Process:**

1. **Two-Pass Analysis**

   - **Pass 1**: Traverse all source files, identify function declarations and calls
   - **Pass 2**: Resolve lazy imports by matching resolved module paths to functions

2. **Caller Resolution**

   - Uses TypeScript's `TypeChecker.getSymbolAtLocation()` to resolve identifiers to declarations
   - Handles aliased symbols (re-exports) via `getAliasedSymbol()`
   - Falls back to name-based matching for certain patterns

3. **Circular Reference Handling**
   - All tree-walking functions use `visited` Set to track seen nodes
   - Prevents infinite recursion in circular dependency scenarios

#### 4. `src/formatter.ts` - Formatter Entry Point

Thin wrapper that delegates to format plugins:

- Auto-registers built-in plugins (markdown, html)
- `generateReport()`: Routes to appropriate plugin
- Maintains backward compatibility with old API

#### 5. `src/plugin/` - Plugin System

**Plugin Registry (`src/plugin/registry.ts`):**

- `register(plugin)`: Register a new format plugin
- `get(name)`: Get plugin by name
- `has(name)`: Check if plugin is registered
- `getAvailableFormats()`: List all registered format names

**Shared Utilities (`src/plugin/shared/`):**

- `calculateImpactScore()`: Calculate impact based on breadth and depth
- `getImpactLevel()`: Convert score to impact level (critical/high/medium/low/none)
- `getImpactLabel()`: Get display label with emoji
- `buildTreeData()`: Build hierarchical tree for visualization
- `buildImpactedItems()`: Build list of impacted items with stats
- `collectEntryPoints()`: Collect test targets (root callers) via BFS
- `refineTestTargets()`: Push down overly-broad convergent roots and deduplicate

#### 6. `src/plugin/format-html/` - HTML Format Plugin

Single-page static HTML report with minimal design. All content is server-rendered — no client-side data fetching or complex interactions.

**Files:**

- `index.ts` — Plugin class, builds data and delegates to templates
- `types.ts` — Type imports
- `styles.ts` — Clean light-theme CSS (~250 lines), system font stack, no external fonts
- `templates.ts` — HTML rendering: tree nodes, changed nodes cards, test targets table, full page layout
- `scripts.ts` — Minimal JS (~15 lines): tree collapse/expand only

**Key Features:**

- **Single-page layout**: All changed nodes with inline dependency trees, visible without clicking
- **Collapsible tree view**: Each changed node shows its callers as a collapsible tree
- **Impact badges**: Color-coded score badges per node
- **Test targets table**: Shows test targets with depth, coverage, and priority

#### 7. `src/plugin/index.ts` - Plugin Exports

Central export point for plugin system:

- Re-exports all plugin types
- Re-exports shared utilities
- Exports built-in plugin instances (`markdownFormatPlugin`, `htmlFormatPlugin`)

#### 8. `src/types.ts` - Type Definitions

Core interfaces for the application:

```typescript
interface CallSite {
  callerId: string;
  line: number;
}
interface LazyImport {
  moduleSpecifier: string;
  line: number;
}
type NodeKind = 'function' | 'method' | 'constructor' | 'accessor'
  | 'class-property' | 'class' | 'variable' | 'enum';
interface FunctionInfo {
  callers: CallSite[];
  definition: { startLine: number; endLine: number };
  kind?: NodeKind;
  lazyImports?: LazyImport[];
}
type CallGraph = Map<string, FunctionInfo>;
type OutputFormat = "markdown" | "html";
```

## Development Conventions

### Code Style

- **Strict TypeScript**: `strict: true` with `noUncheckedIndexedAccess: true`
- **Module System**: ES Modules (`"type": "module"`)
- **Import Style**: Use `.js` extensions for local imports

### Function Naming

- `get*`: Functions that retrieve data
- `build*`: Functions that construct data structures
- `find*`: Functions that search/filter data
- `generate*`: Functions that create formatted output
- `register*`: Functions that register plugins/components
- `refine*`: Functions that iteratively improve/narrow results

### Error Handling

Errors are thrown as `Error` objects with descriptive messages. CLI catches and formats them.

### File ID Format

Functions identified as: `relative/path/to/file.ts:functionName`

## Adding a New Output Format

To add a new format (e.g., JSON):

1. **Create plugin directory:**

   ```
   src/plugin/format-json/
   └── index.ts
   ```

2. **Implement the plugin:**

   ```typescript
   import type { AnalysisResult } from "../../types.js";
   import type { FormatPlugin } from "../types.js";

   export class JsonFormatPlugin implements FormatPlugin {
     readonly name = "json";
     readonly extension = "json";
     readonly contentType = "application/json";

     generate(result: AnalysisResult, maxDepth: number | null): string {
       return JSON.stringify(result, null, 2);
     }
   }

   export const jsonFormatPlugin = new JsonFormatPlugin();
   ```

3. **Register in `src/formatter.ts`:**

   ```typescript
   import { jsonFormatPlugin } from "./plugin/format-json/index.js";
   registerPlugin(jsonFormatPlugin);
   ```

4. **Update type in `src/types.ts`** (if needed for CLI validation)

## Release Process

**Fully automated** - see [RELEASE.md](./RELEASE.md).

### Workflow

1. **Contributor**: Add changeset + PR → merge
2. **CI**: Auto-create "Version Packages" PR
3. **Maintainer**: Merge "Version Packages" PR
4. **CI**: Auto-publish to npm + create git tag

### Setup Required

- `NPM_TOKEN` secret in GitHub repository settings
- That's it! 🎉

## Dependencies

### Production

- `commander`: CLI argument parsing
- `typescript`: TypeScript compiler (bundled)

### Development

- `typescript`: TypeScript compiler
- `esbuild`: Bundler for CLI single-file output
- `@changesets/cli`: Version management
- `@types/node`: Node.js type definitions

## Common Tasks

### Adding a New CLI Option

1. Add option to CLI in `src/index.ts`
2. Pass to `runAnalysis()`
3. Use in analyzer or formatter as needed

### Adding a New Format Plugin

1. Create plugin in `src/plugin/format-<name>/`
2. Implement `FormatPlugin` interface
3. Register in `src/formatter.ts`
4. Update `OutputFormat` type in `src/types.ts`
5. Add tests and documentation

### Fixing a Bug

1. Identify issue in relevant source file
2. Fix in source code
3. Run `pnpm build`
4. Test manually with `pnpm depwalker`

## Edge Cases & Known Behaviors

### What Gets Tracked

| Scenario                       | Behavior                                | Rationale                         |
| ------------------------------ | --------------------------------------- | --------------------------------- |
| Function with no callers       | Shows "No Impact" (⚪)                  | Safe change, no ripple effects    |
| Circular dependency A→B→A      | Stops at cycle, marks circular          | Prevents infinite loops           |
| Function called in 100+ places | Shows "Critical" (🔴) with high score   | High breadth = high risk          |
| Deep call chain (15+ levels)   | Shows "Critical" (🔴) with depth factor | Deep impact = systemic risk       |
| JSX in map/filter              | Each occurrence tracked separately      | Accurate line numbers for changes |

### Performance Edge Cases

| Scenario                          | Impact              | Mitigation                              |
| --------------------------------- | ------------------- | --------------------------------------- |
| 10,000+ functions                 | ~10s analysis time  | Efficient Set-based visited tracking    |
| 100+ test targets from one change | Large report size   | Depth limiting with `--depth` flag; `refineTestTargets()` pushes down broad roots |
| Complex circular dependencies     | Stack overflow risk | All recursive functions track visited   |
| Large monorepo with many files    | Memory usage        | Only analyzes files in tsconfig include |

### Common Misidentifications

The analyzer may occasionally:

- Miss calls through complex dynamic patterns
- Over-count when same function is called multiple times from one caller (shows as multiple call sites)
- Not distinguish between different overloads of same function name
- Include type-only imports in file dependency graph (cosmetic only)

---

_This document should be updated when project structure or conventions change._

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/razrinn)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/razrinn)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
