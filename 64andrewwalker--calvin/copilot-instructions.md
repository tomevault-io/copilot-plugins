## calvin

> > For AI coding assistants working on this codebase.

# Calvin Development Guide

> For AI coding assistants working on this codebase.

## What is Calvin?

Calvin is a PromptOps compiler. It takes a `.promptpack/` directory of Markdown files and compiles them into platform-specific outputs for Claude Code, Cursor, VSCode, Antigravity, and Codex.

**Core command**: `calvin deploy` - compiles and writes outputs to project.

**Documentation**: https://64andrewwalker.github.io/calvin/

## Quick Start for Developers

```bash
# Build
cargo build

# Test (must pass before committing)
cargo test

# Format (required)
cargo fmt

# Lint
cargo clippy
```

## Architecture

Calvin uses **Clean Architecture** with 4 layers:

```
Presentation → Application → Domain ← Infrastructure
```

### Directory Structure

```
src/
├── main.rs              # CLI entry point
├── lib.rs               # Library exports
├── domain/              # Core business logic (no I/O)
│   ├── entities/        # Asset, OutputFile, Lockfile
│   ├── services/        # Compiler, Planner, OrphanDetector
│   ├── policies/        # ScopePolicy, SecurityPolicy
│   ├── value_objects/   # Scope, Target, Hash
│   └── ports/           # Trait definitions
├── application/         # Use cases (orchestration)
│   ├── deploy/          # DeployUseCase
│   ├── watch/           # WatchUseCase, file watching
│   ├── check.rs         # CheckUseCase
│   └── diff.rs          # DiffUseCase
├── infrastructure/      # External integrations
│   ├── adapters/        # Claude, Cursor, VSCode, etc.
│   ├── sync/            # Local/Remote file sync
│   └── fs/              # FileSystem implementations
├── commands/            # CLI command handlers
├── ui/                  # Terminal UI components
├── presentation/        # CLI definitions, factories
├── config/              # Config loading
├── security/            # Security checks
├── models.rs            # PromptAsset, Frontmatter
└── parser.rs            # YAML frontmatter parsing
```

## Key Documents

| Document | When to Read |
|----------|--------------|
| `docs/architecture.md` | Understanding design goals |
| `docs/architecture/layers.md` | Understanding layer responsibilities |
| `docs/architecture/directory.md` | Finding where code lives |
| `docs/api/frontmatter.md` | Working with source file format |
| `docs/target-platforms.md` | Adding new platform adapters |
| `spec.md` | Original product specification |

## Development Principles

1. **TDD always** - Write tests first, then implement. All features must have tests before merging.

2. **No god objects** - Keep files under 400 lines. Large files require `calvin-no-split` justification.

3. **Domain purity** - `domain/` never imports `infrastructure/` or `application/`. It defines traits; others implement.

4. **Update docs** - When changing behavior, update relevant docs in `docs/`. Architecture changes → `docs/architecture/`. API changes → `docs/api/`.

5. **Suggest refactoring** - If a change would benefit from restructuring, say so. Don't silently accumulate debt.

6. **Minimal changes** - Solve the problem at hand. Don't add features that weren't requested.

7. **Security by default** - Never weaken security. Deny lists are mandatory, not optional.

8. **Never fail silently** - Invalid input should warn users, not silently fall back to defaults. A typo like `strct` vs `strict` should produce a helpful message, not silent misbehavior.

## Common Tasks

### Adding a new CLI command

1. Add enum variant to `src/presentation/cli.rs`
2. Add handler in `src/commands/`
3. Add match arm in `src/main.rs`
4. Add tests

### Adding a new platform adapter

1. Create `src/infrastructure/adapters/<platform>.rs`
2. Implement `TargetAdapter` trait from `domain/ports/`
3. Register in `infrastructure/adapters/mod.rs`
4. Add `Target` variant in `domain/value_objects/target.rs`
5. Add tests

### Modifying deploy behavior

1. Read `src/application/deploy/use_case.rs` (main orchestration)
2. Check `domain/services/planner.rs` for conflict detection
3. Check `domain/services/compiler.rs` for output generation
4. Run `cargo test deploy` after changes

## Testing

```bash
# All tests
cargo test

# Specific module
cargo test domain::services::compiler

# With output
cargo test -- --nocapture

# Integration tests
cargo test --test cli_deploy_cleanup
```

## Pre-commit Checklist

1. `cargo fmt`
2. `cargo clippy` - no warnings
3. `cargo test` - all pass
4. Files under 400 lines (or marked with `calvin-no-split`)

## Code Style

- Use `PathBuf` for file paths, not strings
- Error handling: `anyhow::Result` for CLI, custom errors for domain
- Prefer iterators over loops
- Document public APIs with `///`

## TDD Workflow

```bash
# 1. Write failing test
cargo test my_new_feature -- --nocapture
# Expected: FAIL

# 2. Implement minimum code to pass
# ...edit code...

# 3. Run test again
cargo test my_new_feature
# Expected: PASS

# 4. Refactor if needed, keep tests green
```

For TDD session examples, see `docs/archive/tdd-session-*.md`.

## Documentation Sync

When you change code, update corresponding docs:

| Change Type | Update |
|-------------|--------|
| New CLI option | `docs/command-reference.md` |
| Config change | `docs/configuration.md` |
| New adapter | `docs/target-platforms.md` |
| Architecture | `docs/architecture/*.md` |
| API change | `docs/api/changelog.md` |

## Critical Files

Files that affect the whole system - change with extra care:

- `src/lib.rs` - Public API exports
- `src/main.rs` - CLI entry point
- `domain/ports/*.rs` - Trait contracts
- `Cargo.toml` - Dependencies

## Asking for Help

If stuck, check:

1. `docs/architecture/` for design decisions
2. `docs/archive/` for historical context
3. `docs/guides/pitfall-mitigations.md` for known issues
4. Existing tests for usage examples

<skills_system priority="1">

## Available Skills

<!-- SKILLS_TABLE_START -->
<usage>
When users ask you to perform tasks, check if any of the available skills below can help complete the task more effectively. Skills provide specialized capabilities and domain knowledge.

How to use skills:
- Invoke: Bash("openskills read <skill-name>")
- The skill content will load with detailed instructions on how to complete the task
- Base directory provided in output for resolving bundled resources (references/, scripts/, assets/)

Usage notes:
- Only use skills listed in <available_skills> below
- Do not invoke a skill that is already loaded in your context
- Each skill invocation is stateless
</usage>

<available_skills>

<skill>
<name>self-critique</name>
<description>Progressive self-questioning for agents before claiming work is complete.</description>
<location>project</location>
</skill>

</available_skills>
<!-- SKILLS_TABLE_END -->

</skills_system>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/64andrewwalker)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/64andrewwalker)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
