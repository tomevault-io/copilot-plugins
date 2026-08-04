## ngcompass

> This file provides guidance to OpenAI Codex and other AI coding agents when working in this repository. See CLAUDE.md for the full reference — this file contains the same essential guidance.

# AGENTS.md

This file provides guidance to OpenAI Codex and other AI coding agents when working in this repository. See CLAUDE.md for the full reference — this file contains the same essential guidance.

## Commands

```bash
pnpm build              # dev build (all packages)
pnpm build:prod         # production build (minified, required before publish)
pnpm test               # run all tests
pnpm typecheck          # type-check all packages
pnpm prerelease:check   # smoke + validate before publishing
pnpm release:beta       # full beta publish pipeline
pnpm clean              # clear turbo and node_modules caches
```

Run tests for a single package:

```bash
pnpm --filter @ngcompass/engine vitest run
pnpm --filter @ngcompass/rules vitest run src/rules/reactivity/rxjs-no-subscribe-in-component.rule.test.ts
```

## Monorepo Structure

Turborepo + pnpm workspaces. All packages live in `packages/`. Root `package.json` is `private: true` and is never published.

| Package              | Purpose                                                             |
| -------------------- | ------------------------------------------------------------------- |
| `packages/cli`       | Binary (`ngcompass`), command orchestration                         |
| `packages/config`    | Config discovery, validation, normalization                         |
| `packages/scanner`   | File discovery (git/glob), file-list cache                          |
| `packages/rules`     | All built-in rules, presets, RuleRegistry                           |
| `packages/planner`   | Task graph, content-addressed task IDs, incremental filtering       |
| `packages/engine`    | Single-pass AST execution, worker pool, type-aware chunking         |
| `packages/ast`       | Oxc TS parser, Angular HTML parser, CSS parser, stream types        |
| `packages/cache`     | Multi-layer cache (config, file, plan, result, analysis, meta)      |
| `packages/reporters` | Console, JSON, SARIF, HTML reporters                                |
| `packages/common`    | Shared types: `RuleContext`, `RuleResult`, `AnalysisResult`, `Task` |

`packages/site` is docs only — excluded from build and publish.

## Architecture in One Paragraph

The CLI orchestrates a strict pipeline: load config → discover files → resolve rules → build execution plan → run analysis → emit reports. The **planner** converts files + rules into content-addressed tasks (`taskId` = hash of file content + rule options). Warm runs skip tasks whose `taskId` already exists in the result cache — sometimes the entire analysis is skipped via a full-analysis cache hit. The **engine** splits tasks into syntax-only (worker pool, parallel) and type-aware (main thread, chunked TypeScript Programs). Rules are passive stream observers registered in `RuleRegistry`; they never do their own AST traversal.

Full architecture reference: `docs/architecture.md`.

## Adding a Rule

1. Create `packages/rules/src/rules/<domain>/<rule-name>.rule.ts`
2. Use a factory from `@ngcompass/engine` matching the stream type you need:
   `createComponentRule`, `createTemplateExpressionRule`, `createCallExpressionRule`, etc.
3. Declare `RuleMetadata` — set `dependencyType` to `syntax`, `type-aware`, or `project-context`
4. Register in `packages/rules/src/registry/register-all.ts`
5. Add to the appropriate preset in `packages/rules/src/presets/`

Rules receive pre-filtered nodes in `handle(node, context)`. They must be stateless, allocation-minimal, and O(1) per node.

## Key Conventions

- Use typed `Result<T>` objects for errors, not thrown exceptions (except unrecoverable startup failures).
- Progress/debug output → `stderr`. Machine-readable output (JSON, SARIF) → `stdout`.
- Cross-package imports always use package names (`@ngcompass/common`), never relative paths across package boundaries.
- TypeScript strict mode, `module: "Node16"`, ES2022 target. Include file extensions in ESM imports.
- Do not lower coverage thresholds in `vitest.config.ts` — they are intentionally minimal during beta.
- When changing `RuleResult` shape or planner task format, bump the relevant `CACHE_SCHEMA_VERSION` or `PLAN_CACHE_VERSION` constant in `@ngcompass/cache`.

## License

PolyForm Shield 1.0.0. Free for any use except building a competing product. See `LICENSE`.

---

# Coding Standards

These standards exist because this is a performance-sensitive static analyzer. Violating them creates measurable regressions, not stylistic noise.

## Type Discipline

- **Never use `any`.** Use `unknown` and narrow with type guards.
- **Never use `as` casts** except for `as const` or narrowing from `unknown` after a runtime check. Use type guards.
- **Prefer `interface` for object shapes**, `type` for unions/intersections/utilities.
- **Mark fields and arrays `readonly`** when not mutated after construction.
- **No `Record<string, any>` or `object`** as a parameter type — define a real interface.
- **Explicit return types on every exported function.**
- **No optional parameters with side-effect defaults** — pass values explicitly.

## Error Handling

- **Domain errors return `Result<T, E>`.** Don't throw for expected failures.
- **Throw only for programmer errors** (unreachable states, broken invariants).
- **Never swallow errors silently.** No empty `catch {}`.
- **No try/catch around code that cannot throw** — it adds noise and slows V8.
- **Cache deserialization corruption is the one place** that catches and rebuilds.

## Performance (Hot Path: Engine, Planner, Rule Handlers)

| Rule                                                    | Why                                         |
| ------------------------------------------------------- | ------------------------------------------- |
| No allocations inside AST traversal callbacks           | Single-pass engine visits millions of nodes |
| Avoid `Object.keys`/`values`/`entries` in hot loops     | They allocate intermediate arrays           |
| Cache regex objects at module scope                     | `new RegExp` per call is expensive          |
| Use `Map` for keyed string lookups, not plain objects   | Predictable O(1), supports `.clear()`       |
| Bounded caches must evict — see `CACHE_LIMIT` constants | Unbounded caches leak in long-running CI    |
| No `JSON.parse`/`stringify` in rule handlers            | Belongs at cache boundaries only            |
| Use classical `for` loops in hot paths                  | V8 optimizes them most aggressively         |

## Clean Code

- **Function length** ≤30 lines of executable code. Longer = doing multiple things.
- **Cyclomatic complexity** ≤10. Use early returns and lookup tables.
- **Pure functions by default.** Side effects live at orchestration layers (commands, runner), not in rules/planners/transforms.
- **Naming**:
  - Functions: verb + noun (`resolveRules`). No `do`, `handle`, `process`, `manage`.
  - Booleans: `is*`, `has*`, `should*`, `can*`.
  - No `Async` suffix — `Promise<T>` already says it.
  - Magic numbers extracted to module-level constants.
- **No dead code, no commented-out code.** Git has history.

## SOLID (Applied)

- **S** — Each package has one responsibility (see Monorepo Structure). Don't move cache logic into engine, planner logic into rules, etc.
- **O** — New rules extend the registry without modifying engine. New reporters extend `getReporter(format)` without modifying analysis.
- **L** — All `Reporter` implementations must work everywhere a `Reporter` is expected.
- **I** — Don't expand narrow interfaces (`Result<T>`) for one consumer — create a new type.
- **D** — Engine depends on `RuleRegistry` interface, not concrete rules. Cache depends on driver contracts, not specific backends.

## Clean Architecture Layers

Dependencies may only flow inward:

```
CLI → config/scanner/planner/engine/reporters → rules/cache/ast → common
```

Forbidden:

- `common` importing from any other package
- `engine` importing concrete rules
- `rules` importing from `planner` or `cli`
- Reporters touching the filesystem directly
- Anything outside CLI calling `process.exit`

## No Comments In Code

- **Code comments are forbidden** in source, test, script, and configuration files. Do not add JSDoc, `@fileoverview` blocks, block comments, line comments, section banners, TODO comments, commented-out code, or disable comments.
- **Remove existing comments from any code file you modify.** When touching a source, test, script, or configuration file, erase comments from that entire file in the same change.
- **Use code structure instead of comments.** If something needs explanation, improve names, extraction, types, or control flow until the code explains itself.
- **Markdown prose is exempt.** Documentation files may use normal prose, but embedded code examples should still avoid comments.

## Node.js Rules

- **ESM only.** No `require()` in source. Imports include `.js` extension (Node16 module resolution).
- **No synchronous `fs`** in hot paths. Use `fs/promises`. Sync I/O is acceptable at startup.
- **No `process.exit()`** outside the CLI command layer.
- **Worker threads for CPU-bound parallel work.** Don't block the event loop.
- **`AbortController` + `AbortSignal`** for cancellation. Workers and type-aware chunks check `signal.aborted` at boundaries.
- **Read environment variables once at startup** into a typed config object. No `process.env.X` in library code.
- **Remove `EventEmitter` listeners explicitly.** Workers, watchers, signal handlers all leak otherwise.

## Monorepo Discipline

- **Import only from package public exports.** Never reach into another package's `src/` or `dist/`.
- **No circular package dependencies.** `pnpm turbo build --dry` catches them.
- **Workspace deps use `workspace:*`** — never pin to a specific version.
- **Add a sibling package to `dependencies`** the moment you import from it. `validate:packages` enforces this.
- **Version bumps are atomic across all 10 packages.**
- **One concern per package.** Features spanning 3 packages? At least one is in the wrong place.

## Testing

- **Test the public API**, not internals.
- **No mocks of types you own** — build real `RuleContext` instances.
- **Each test isolates its temp dirs** via `os.tmpdir()` + unique subdirs.
- **Integration tests touch real I/O** for cache/scanner/engine. Don't stub.
- **One concept per `it()` block.** Multiple `expect()` calls fine if testing the same concept.
- **Test names describe behavior**: `it('skips ignored files when gitignore matches')`, not `it('scan() works')`.

## Common LLM Mistakes — Don't

- **Don't abstract before needed.** Three similar lines beats a premature helper.
- **Don't add defensive checks for impossible states.** TypeScript already guarantees types.
- **Don't wrap working code in try/catch "just in case."**
- **Don't add disable comments.** Fix the issue or adjust configuration instead.
- **Don't refactor unrelated code** in a feature change. Separate PRs.
- **Don't add new npm dependencies casually.** Justify each one. Prefer Node built-ins.
- **Don't generate dead exports or unused parameters.** Strict TypeScript will reject them.
- **Don't add explanatory code comments.** Improve the code until the explanation is unnecessary.
- **Don't write functions over ~50 lines** without extracting helpers.

## When in Doubt

1. Read equivalent existing code. Match its style and shape.
2. If two patterns conflict, prefer the newer one.
3. If still unclear, ask before writing.

---
> Source: [RoadmapDevelop/ngcompass](https://github.com/RoadmapDevelop/ngcompass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
