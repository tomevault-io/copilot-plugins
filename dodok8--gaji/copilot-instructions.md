## gaji

> **gaji** is a Rust CLI tool that enables type-safe GitHub Actions workflow authoring in TypeScript. Users write workflows in TypeScript with full type safety and autocomplete, and gaji compiles them to YAML files that GitHub Actions understands.

# CLAUDE.md

## Project Overview

**gaji** is a Rust CLI tool that enables type-safe GitHub Actions workflow authoring in TypeScript. Users write workflows in TypeScript with full type safety and autocomplete, and gaji compiles them to YAML files that GitHub Actions understands.

- **Repository**: https://github.com/dodok8/gaji
- **Language**: Rust (edition 2021)
- **License**: MIT
- **Version**: defined in `Cargo.toml`

## Quick Reference Commands

```bash
# Build the project
cargo build

# Build release binary (optimized for size)
cargo build --release

# Run all tests
cargo test --all-features

# Lint check
cargo clippy --all-targets --all-features -- -D warnings

# Format check
cargo fmt --all --check

# Format code
cargo fmt --all
```

## Writing Docs

Before Writing docs, See <https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing> and avoid these patterns.

## Project Structure

```
src/
├── main.rs          # CLI entry point, command routing via clap
├── lib.rs           # Public module exports
├── cli.rs           # CLI command/argument definitions (clap derive)
├── config.rs        # Configuration loading (gaji.config.ts / gaji.config.local.ts)
├── builder.rs       # TypeScript → YAML workflow compilation pipeline
├── executor.rs      # JavaScript execution via QuickJS runtime
├── cache.rs         # Action metadata caching (.gaji-cache.json)
├── fetcher.rs       # GitHub API client for fetching action.yml files
├── watcher.rs       # File system watcher for dev mode (notify crate)
├── parser/
│   ├── mod.rs       # Parser module interface
│   ├── ast.rs       # AST visitor for extracting action references
│   └── extractor.rs # Action reference extraction from TypeScript code
├── generator/
│   ├── mod.rs       # Type generation orchestration
│   ├── types.rs     # TypeScript type definition (.d.ts) generation
│   └── templates.rs # Hardcoded type templates (base types, runtime)
└── init/
    ├── mod.rs       # Project initialization & state detection
    ├── interactive.rs  # Interactive init mode (dialoguer)
    ├── migration.rs    # YAML → TypeScript migration
    └── templates.rs    # Template files for new projects

tests/
└── integration.rs   # End-to-end builder→executor→YAML pipeline tests

workflows/           # gaji's own CI workflows (self-dogfooding)
├── audit.ts
├── ci.ts
├── release-plz.ts
├── release.ts
├── update-workflows.ts
└── vitepress.ts

npm/                 # NPM distribution wrapper
├── gaji/            # Main npm package (postinstall downloads binary)
└── platform-*/      # Platform-specific binary packages
```

## Architecture

The core pipeline is: **TypeScript → Parse → Execute → YAML**

1. **Parser** (`parser/`): Uses oxc to parse TypeScript and extract `getAction("owner/repo@version")` calls via AST visitor pattern
2. **Fetcher** (`fetcher.rs`): Downloads `action.yml` from GitHub for referenced actions
3. **Generator** (`generator/`): Generates TypeScript type definitions (`.d.ts`) from action metadata
4. **Executor** (`executor.rs`): Strips TypeScript types with oxc, bundles with runtime JS, executes in QuickJS
5. **Builder** (`builder.rs`): Orchestrates the full pipeline, converts JSON output to YAML, writes workflow files

## Key Design Patterns

- **Builder pattern**: `WorkflowBuilder`, `Job`, `Workflow`, `WorkflowCall`, `ActionRef`, `Action`, `NodeAction`, `DockerAction`
- **Callback builder pattern**: `StepBuilder` accumulates step context (`_ctx`), `JobBuilder` accumulates job output context — callbacks receive previous outputs for type-safe chaining
- **Visitor pattern**: `ActionRefExtractor` traverses oxc AST nodes
- **Error handling**: `anyhow::Result<T>` with `?` propagation throughout; `thiserror` for typed errors
- **Async**: Tokio runtime for all I/O-bound operations (HTTP, filesystem)
- **Configuration hierarchy**: env vars > `gaji.config.local.ts` > `gaji.config.ts` > defaults

## Code Conventions

- **Modules**: `snake_case` (e.g., `type_generator`)
- **Types/Structs**: `PascalCase` (e.g., `WorkflowBuilder`, `TypeGenerator`)
- **Functions/Methods**: `snake_case` (e.g., `extract_action_refs`, `build_all`)
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `CACHE_FILE`, `CONFIG_FILE`)
- **Action references**: `owner/repo@version` format (e.g., `actions/checkout@v5`)
- **Tests**: Inline with `#[cfg(test)]` blocks in each module
- **CLI output**: Uses `colored` crate for colored terminal output with emoji prefixes

## Configuration Files

| File | Purpose | Committed |
|------|---------|-----------|
| `Cargo.toml` | Rust dependencies and release profile | Yes |
| `gaji.config.ts` | Project config (dirs, watch settings, build options) | Yes |
| `gaji.config.local.ts` | Private config (GitHub token, custom API URL) | No |
| `mise.toml` | Dev environment (deno latest, node LTS) | Yes |
| `release-plz.toml` | Automated version bumping config | Yes |

## CI/CD

CI workflows live in `.github/workflows/` and are **generated from** `workflows/ci.ts` using gaji itself (self-dogfooding).

**PR checks** (`.github/workflows/pr.yml`):
- `cargo test --all-features`
- `cargo clippy --all-targets --all-features -- -D warnings`
- `cargo fmt --all --check`

All three checks must pass before merging.

## Testing

```bash
# Run all tests (unit + integration)
cargo test --all-features

# Run a specific test
cargo test test_name

# Run tests in a specific module
cargo test module_name::
```

- **Unit tests**: Inline `#[cfg(test)]` in each source file
- **Integration tests**: `tests/integration.rs` — full pipeline tests
- **Test utilities**: `tempfile` crate for temporary directories in tests

## Key Dependencies

| Crate | Purpose |
|-------|---------|
| `clap` (derive) | CLI argument parsing |
| `tokio` (full) | Async runtime |
| `oxc_*` | TypeScript parsing, AST, transformation, codegen |
| `rquickjs` | Embedded JavaScript execution engine |
| `reqwest` | HTTP client for GitHub API |
| `serde_yaml` / `serde_json` | Serialization |
| `notify` | Filesystem watching |
| `colored` | Terminal colors |
| `indicatif` | Progress bars |
| `dialoguer` | Interactive terminal prompts |

## Release Profile

The release binary is optimized for size (important for npm distribution):
- `opt-level = "z"` — size optimization
- `lto = true` — link-time optimization
- `codegen-units = 1` — full optimization
- `strip = true` — strip debug symbols

## Common Workflows

**Adding a new CLI command**: Define variant in `Commands` enum in `src/cli.rs`, add handler function in `src/main.rs`, match in `main()`.

**Adding support for a new action.yml field**: Update `fetcher.rs` for parsing, `generator/types.rs` for type generation, and `generator/templates.rs` if base types need changes.

**Adding a new action type** (like `NodeAction`, `DockerAction`): Add runtime class to `generator/templates.rs` (`JOB_WORKFLOW_RUNTIME_TEMPLATE`), add TypeScript interfaces to `BASE_TYPES_TEMPLATE`, add type declaration in `CLASS_DECLARATIONS_TEMPLATE`. Output type "action" writes to `.github/actions/<id>/action.yml`.

**Adding a new job type**: Extend the `Job` class. Job subclasses inherit `steps()`, `outputs()`, and `toJSON()` to produce standard `JobDefinition` output. No separate runtime class is needed — use JavaScript class inheritance (`class MyJob extends Job`).

**Modifying the build pipeline**: Core logic is in `builder.rs` (orchestration) and `executor.rs` (JS execution). The executor strips types with oxc and runs the result in QuickJS.

## Runtime Class Hierarchy

The TypeScript runtime classes in `generator/templates.rs` follow this hierarchy:

| Class | Purpose | Output Type | Build Target |
|-------|---------|-------------|--------------|
| `StepBuilder` | Accumulates steps with `_ctx` for output chaining | Internal | Used by `Job.steps()` / `Action.steps()` |
| `JobBuilder` | Accumulates jobs with `_ctx` for output chaining | Internal | Used by `Workflow.jobs()` |
| `Job` | Standard workflow job | `JobDefinition` | Part of `Workflow` |
| `Workflow` | Complete workflow | `WorkflowDefinition` | `.github/workflows/<id>.yml` |
| `Action` | Reusable composite action | `action.yml` (composite) | `.github/actions/<id>/action.yml` |
| `NodeAction` | Node.js-based action | `action.yml` (node) | `.github/actions/<id>/action.yml` |
| `DockerAction` | Docker-based action | `action.yml` (docker) | `.github/actions/<id>/action.yml` |
| `WorkflowCall` | Reusable workflow call (no steps) | `{ uses: ... }` | Part of `Workflow` |
| `ActionRef` | Reference to local action | `{ uses: "./.github/actions/<id>" }` | Step in a `Job` |

## Files to Never Edit Manually

- `.github/workflows/*.yml` — generated by gaji from `workflows/*.ts`
- `.github/actions/` — generated action definitions
- `generated/` directory — auto-generated type definitions
- `.gaji-cache.json` — auto-managed cache
- `npm/platform-*/bin/` — populated by CI release process

---
> Source: [dodok8/gaji](https://github.com/dodok8/gaji) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
