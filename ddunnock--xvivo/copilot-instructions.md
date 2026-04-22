## xvivo

> Skill test harness with autonomous prompt optimization. Two modes: `xvivo run` (fast pass/fail) and `xvivo optimize` (Karpathy loop for iterative skill improvement).

# xvivo

Skill test harness with autonomous prompt optimization. Two modes: `xvivo run` (fast pass/fail) and `xvivo optimize` (Karpathy loop for iterative skill improvement).

## Commands

```bash
npm run dev          # run with tsx (ESM, no compile step)
npm run build        # tsc → dist/
npm run test         # vitest run
npm run test:watch   # vitest watch
npm run lint         # biome check + tsc --noEmit
```

## Tech stack

- **TypeScript** (strict, ESM-only) — `"type": "module"` in package.json
- **Claude Agent SDK** (`@anthropic-ai/claude-agent-sdk`) — agent invocation via `query()`
- **Ink** + **Pastel** — React-based TUI rendering, CLI command routing
- **Vitest** — test runner (ESM-native, no Jest)
- **execa** — subprocess execution for Python scripts
- **Zod** — runtime schema validation for test specs and configs
- **simple-git** — git operations for the optimization loop's keep/revert cycle
- **PyTorch** + **sentence-transformers** — ML-based evaluators (Class C), runs as Python subprocess
- **Biome** — linting + formatting (replaces ESLint + Prettier)

## Architecture

- `src/cli/` — Ink components and Pastel commands (`run`, `optimize`, `train`, `models`)
- `src/runner/` — test discovery, orchestration, result collection
- `src/evaluators/` — Class A (deterministic), B (LLM-judge), C (ML-model) evaluators
- `src/optimizer/` — Karpathy loop: mutation operators, keep/revert, results logging
- `src/sandbox/` — temp directory lifecycle, script copying, fixture writing, output collection
- `src/parsers/` — markdown section parser, test spec YAML loader
- `src/types/` — shared Zod schemas and TypeScript types
- `scripts/ml/` — Python ML evaluator scripts (embedding similarity, classifier, anomaly, calibrator)
- `models/` — trained ML model checkpoints (gitignored except metadata)
- `.claude/references/` — detailed best practices per technology (read on demand, not in every session)

## Conventions

- Named exports only, no default exports
- Async/await over raw promises — never `.then()` chains
- All Python interaction goes through `execa` with JSON on stdin/stdout — never import Python from Node
- Evaluators return `EvalResult` (see `src/types/eval.ts`) — always `{ passed, score, details }`
- Test specs are YAML with Zod validation — malformed specs fail at load time, not runtime
- The optimization loop NEVER modifies evaluation code — strict separation of concerns
- Git operations use simple-git, not shell `git` commands
- All exported symbols (functions, schemas, types, classes, interfaces) MUST have JSDoc comments — see `.claude/references/jsdoc-conventions.md`
- All new features and bug fixes MUST use Test-Driven Development (vertical slices, not horizontal) — invoke the `/tdd` skill and see `.claude/references/tdd-workflow.md`

## IMPORTANT

- YOU MUST use `.js` extensions in all TypeScript import paths (ESM resolution requires it)
- YOU MUST keep Ink components in `.tsx` files, all other TypeScript in `.ts` files
- NEVER use `require()` — this is an ESM-only project
- NEVER use `localStorage` or browser APIs — this is a Node.js CLI application
- NEVER put ML model files (`.pt`, `.pkl`) in git — only `metadata.json` is committed
- When adding a new evaluator, add it to all three places: the type union in `src/types/eval.ts`, the registry in `src/evaluators/index.ts`, and the Zod schema in `src/types/spec.ts`

## Reference docs

Read these BEFORE working on the relevant subsystem. They contain antipatterns that will waste your time.

- `.claude/references/ink-and-pastel.md` — Ink component patterns, Static for streaming output, TSX gotchas
- `.claude/references/agent-sdk.md` — query() streaming, message types, hooks, sessions, tool permissions
- `.claude/references/esm-typescript.md` — ESM module resolution, .js extensions, tsx runner, vitest config
- `.claude/references/python-subprocess.md` — execa patterns, JSON stdin/stdout protocol, venv management
- `.claude/references/ml-evaluators.md` — PyTorch model lifecycle, embedding caching, graceful degradation
- `.claude/references/optimization-loop.md` — Karpathy loop mechanics, mutation operators, results logging, git safety
- `.claude/references/jsdoc-conventions.md` — JSDoc requirements for exported symbols, patterns for schemas vs functions vs types
- `.claude/references/tdd-workflow.md` — TDD vertical slice workflow, mocking boundaries, test quality checklist, invoke `/tdd` skill

<!-- GSD:project-start source:PROJECT.md -->
## Project

**xvivo**

A test harness and autonomous prompt optimizer for AI instruction files (skills, commands, agents, prompts). It parses instruction files into testable units — YAML frontmatter, XML blocks, markdown sections — then runs those units through a three-class evaluation pipeline (deterministic, LLM-judge, ML-model) to produce fast pass/fail results. In optimize mode, it iteratively mutates individual sections using a Karpathy-style hill-climbing loop, keeping changes that improve the evaluation score and reverting those that don't. Intended for open-source release.

**Core Value:** Fast, reliable evaluation of prompt quality at the section level — so prompt authors get the same tight feedback loop that software engineers get from unit tests.

### Constraints

- **ESM-only**: `"type": "module"` — no `require()`, `.js` extensions in all imports
- **Node.js >= 25.8.1**: enforced in package.json engines
- **CPU-first ML**: all ML models must work on CPU; GPU is optional optimization
- **5s inference budget**: any ML model exceeding 5s per evaluation falls back to Class B
- **500MB disk / 1GB RAM**: sentence embedding model size limits
- **Determinism**: Class A and C evaluators must produce identical results across runs; only Class B is non-deterministic
- **Separation of concerns**: optimization agent never modifies evaluation code; evaluation harness never modifies skills
- **JSON stdin/stdout**: all TypeScript↔Python communication via subprocess with JSON protocol
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Languages
- TypeScript 5.9.3 (strict mode, ESM-only) - All application code in `src/`
- Target: ES2022, Module: NodeNext
- Python 3 - ML evaluator scripts in `scripts/ml/` (not yet scaffolded)
## Runtime
- Node.js >= 25.8.1 (enforced via `engines` in `package.json`)
- Python 3 venv at `.xvivo/ml-venv/` (for ML evaluators)
- npm
- Lockfile: `package-lock.json` (expected)
## Frameworks
- Ink 6.8.0 - React-based terminal UI rendering (`src/cli/`)
- Pastel 4.0.1 - CLI command routing built on Ink
- React 19.2.4 - JSX rendering engine for Ink components
- `@anthropic-ai/claude-agent-sdk` 0.2.81 - Agent invocation via `query()`
- `@anthropic-ai/sdk` 0.80.0 - Anthropic API client
- Vitest 4.1.1 - Test runner (ESM-native)
- ink-testing-library 4.0.0 - Ink component testing
- TypeScript compiler (`tsc`) - Compiles to `dist/`
- tsx 4.21.0 - Dev runner (ESM, no compile step)
- Biome 2.0 - Linting and formatting (replaces ESLint + Prettier)
## Key Dependencies
- `@anthropic-ai/claude-agent-sdk` 0.2.81 - Core agent invocation
- `ink` 6.8.0 + `pastel` 4.0.1 - TUI framework
- `zod` 4.3.6 - Runtime schema validation for test specs and configs
- `execa` 9.6.1 - Subprocess execution (Python ML scripts)
- `simple-git` 3.33.0 - Git operations for optimization loop keep/revert
- `js-yaml` 4.1.1 - YAML parsing for test spec files
- `ink-ui` 0.4.0 - Pre-built Ink UI components
- `lefthook` 2.1.4 - Git hooks manager
- `@commitlint/cli` 20.5.0 - Conventional commit enforcement
- `@sindresorhus/tsconfig` 8.1.0 - Base TypeScript config
## Configuration
- Strict mode with `noUnusedLocals`, `noUnusedParameters`
- JSX: `react-jsx` with `jsxImportSource: react`
- Output: `dist/` directory
- Source: `src/` only (tests excluded from compilation)
- Formatter: 2-space indent, 100-char line width, double quotes, semicolons always
- Linter: recommended rules + `noUnusedVariables`, `noUnusedImports`, `noExplicitAny` as errors
- Import organization enabled
- Ignores: `dist/`, `node_modules/`, `models/`, `.xvivo/ml-venv/`, `*.py`
- Environment: node
- Test location: `tests/**/*.test.{ts,tsx}`
- Globals: true (no explicit imports needed for `describe`/`it`/`expect`)
- Pool: forks, timeout: 30s, hook timeout: 15s
- Pre-commit: Biome check, TypeScript typecheck, Python syntax check, ban `.pt`/`.pkl` files, ban `require()`, ban `console.log()` in TSX, enforce `.js` import extensions
- Commit-msg: Conventional commits via commitlint (custom type `xvivo` allowed)
- Pre-push: Run vitest
## Build & Run Commands
## Binary Entry Point
- CLI binary: `dist/cli/index.js` (registered as `xvivo` in `package.json` bin)
## Platform Requirements
- Node.js >= 25.8.1
- Python 3 (for ML evaluators, optional)
- Node.js CLI tool, distributed via npm
- No server deployment - runs locally
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

## Module System
- Use `import`/`export` exclusively. `require()` is banned (enforced by lefthook pre-commit hook).
- All relative imports MUST use `.js` extensions (ESM resolution). Enforced by lefthook `import-extensions` hook.
- Named exports only, no default exports. Biome rule `noDefaultExport` is set to `"off"` but project convention (per `CLAUDE.md`) is named exports only.
- Use `import type` for type-only imports. Biome rule `useImportType: "error"` enforces this.
## File Extensions
- `.ts` for all TypeScript source files.
- `.tsx` for Ink (React-based TUI) components only.
- `.js` for config files that need ESM default exports (e.g., `commitlint.config.js`).
- Never commit `.pt` or `.pkl` ML model files (enforced by lefthook hook).
## TypeScript Configuration
- Strict mode enabled (`"strict": true` in `tsconfig.json`).
- Target: `ES2022`, Module: `NodeNext`, Resolution: `NodeNext`.
- `noUnusedLocals: true`, `noUnusedParameters: true`, `noFallthroughCasesInSwitch: true`.
- JSX configured for React (`"jsx": "react-jsx"`, `"jsxImportSource": "react"`).
- Source root: `src/`, Output: `dist/`.
## Formatting (Biome)
- Indent: 2 spaces
- Line width: 100 characters
- Quotes: double (`"`)
- Semicolons: always
- Trailing commas: always (JS/TS), never (JSON)
- Arrow function parentheses: always
## Linting (Biome)
- Recommended rules enabled.
- `noUnusedVariables: "error"` -- no dead code.
- `noUnusedImports: "error"` -- clean import blocks.
- `noExplicitAny: "error"` -- no `any` type; use `unknown` or proper types.
- `noConsole: "off"` -- console usage is allowed (CLI application).
- Import organization enabled (`organizeImports: true`).
- Ignored paths: `dist/`, `node_modules/`, `models/`, `.xvivo/ml-venv/`, `*.py`.
## Naming Patterns
- Lowercase with hyphens for multi-word: `index.ts`, `index.test.ts`
- Co-located tests in `tests/` directory, not alongside source files
- UPPER_CASE for constants: `VERSION`
- camelCase for functions (per standard TypeScript convention)
## Import Organization
## Error Handling
- Async/await over raw promises. Never use `.then()` chains.
- Evaluators always return `EvalResult` (`{ passed, score, details }`).
- Zod validates specs at load time -- malformed specs fail early, not at runtime.
## Ink/TUI-Specific Rules
- No `console.log()` in `.tsx` files (enforced by lefthook hook). It corrupts Ink terminal state.
- Use `<Text>` components or `console.error()` instead.
- No browser APIs (`localStorage`, etc.) -- this is a Node.js CLI.
## Git Hooks (Lefthook)
- `biome check --write` on staged `.ts/.tsx/.js/.json` files (auto-fixes and re-stages)
- `tsc --noEmit` typecheck
- `python3 -m py_compile` on staged `.py` files
- Block `.pt/.pkl` model binary commits
- Block `require()` in `.ts/.tsx`
- Block `console.log()` in `.tsx`
- Enforce `.js` extensions in relative imports
- Conventional commits via commitlint (`commitlint.config.js`)
- Allowed types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `ci`, `perf`, `style`, `revert`, `xvivo`
- Subject max length: 100 characters
- `vitest run` -- all tests must pass before push
## Python Interaction
- All Python interaction goes through `execa` with JSON on stdin/stdout.
- Never import Python from Node directly.
- Python files are linted with `py_compile` only (not Biome).
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

## Pattern Overview
- Two execution modes sharing the same evaluation engine: `xvivo run` (single-pass) and `xvivo optimize` (iterative Karpathy loop)
- Three-class evaluator hierarchy (A: deterministic, B: LLM-judge, C: ML-model) executed in strict order
- TypeScript orchestrator with Python ML subprocesses communicating via JSON on stdin/stdout
- Strict separation of concerns: optimization agent, evaluation harness, and orchestrator never cross boundaries
## Layers
- Purpose: Parse commands, render TUI dashboards, handle user input
- Location: `src/cli/`
- Contains: Pastel commands (`run.tsx`, `optimize.tsx`, `train.tsx`, `models/check.tsx`), Ink components
- Depends on: Runner, Optimizer
- Used by: End user via `xvivo` binary
- Purpose: Test discovery, spec loading, evaluator dispatch, result collection
- Location: `src/runner/`
- Contains: Test discovery, execution orchestration, early-termination logic
- Depends on: Evaluators, Parsers, Sandbox, Types
- Used by: CLI commands (`run`, `optimize`)
- Purpose: Execute 25 boolean criteria per skill in class order (A -> B -> C)
- Location: `src/evaluators/`
- Contains: Class A (deterministic TS), Class B (LLM-judge via Agent SDK), Class C (ML-model via Python subprocess)
- Depends on: Types, Python scripts in `scripts/ml/`
- Used by: Runner
- Purpose: Karpathy loop — select mutation operator, invoke optimization agent, keep/revert via git
- Location: `src/optimizer/`
- Contains: Mutation operator selection, keep/revert logic, results logging, cost tracking, stopping conditions
- Depends on: Runner (for evaluation), simple-git (for version control), Agent SDK (for optimization agent)
- Used by: CLI `optimize` command
- Purpose: Create isolated temp directories for test execution
- Location: `src/sandbox/`
- Contains: Temp directory lifecycle, script copying, fixture writing, output collection
- Depends on: Node.js `fs/promises`, `os`
- Used by: Runner
- Purpose: Load and validate test specs and skill files
- Location: `src/parsers/`
- Contains: YAML test spec loader, markdown section parser
- Depends on: `js-yaml`, `zod`, Types
- Used by: Runner
- Purpose: Shared Zod schemas and TypeScript types
- Location: `src/types/`
- Contains: `EvalResult` type (`{ passed, score, details }`), test spec schemas, evaluator class unions
- Depends on: `zod`
- Used by: All other layers
## Data Flow
- Git branch is the source of truth for skill state during optimization
- Results files (`results.tsv`, `results-detail.jsonl`) are append-only logs
- ML model state lives in `models/` directory with `metadata.json` for versioning
## Key Abstractions
- Purpose: Universal return type for all evaluator classes
- Defined in: `src/types/eval.ts`
- Pattern: `{ passed: boolean, score: number, details: Record<string, unknown> }`
- Purpose: YAML definition of 25 boolean criteria with evaluator class assignments
- Defined in: `src/types/spec.ts` (Zod schema)
- Loaded by: `src/parsers/`
- Purpose: Constrained change type for the optimization agent
- Types: `add-constraint`, `add-negative-example`, `add-positive-example`, `restructure`, `tighten-language`, `remove-bloat`, `add-counterexample`, `modify-python-script`
- Selection: Weighted random favoring recently successful operators, never repeating consecutively
## Entry Points
- Location: `src/cli/index.tsx` (dev: `tsx`, prod: `dist/cli/index.js`)
- Triggers: `xvivo run`, `xvivo optimize`, `xvivo train`, `xvivo models check`
- Responsibilities: Pastel command routing, Zod option parsing, delegation to business logic
- Location: `src/index.ts`
- Contains: `VERSION` export (currently `0.1.0` — scaffolding stage)
## Error Handling
- Class C model missing/stale/broken -> fall back to Class B (LLM-judge with equivalent rubric)
- Python venv missing -> skip all Class C, warn once
- Agent SDK rate limit -> back off 60s, retry
- Python script crash (non-zero exit) -> return `{ passed: false, score: 0, details: { error: "script_crash" } }`
- Python script timeout -> return `{ passed: false, score: 0, details: { error: "timeout" } }`
- SDK errors never crash the test runner; caught, logged, marked as "error" (distinct from "fail")
## Cross-Cutting Concerns
<!-- GSD:architecture-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd:quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd:debug` for investigation and bug fixing
- `/gsd:execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->

<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd:profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ddunnock) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
