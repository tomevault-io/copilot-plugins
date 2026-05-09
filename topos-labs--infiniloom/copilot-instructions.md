## infiniloom

> Universal guide for AI coding agents (Claude Code, Codex, Gemini, Cursor, etc.) working on the Infiniloom project.

# AGENTS.md

Universal guide for AI coding agents (Claude Code, Codex, Gemini, Cursor, etc.) working on the Infiniloom project.

## Project Overview

**Infiniloom** is a high-performance repository context generator for Large Language Models, built in pure Rust. It transforms codebases into optimized formats for Claude, GPT-4o/GPT-5, Gemini, and other LLMs.

Key capabilities:

- AST-based symbol extraction using Tree-sitter (23 languages)
- PageRank-based symbol importance ranking
- Model-specific output formats (XML for Claude, Markdown for GPT, YAML for Gemini)
- Automatic secret detection and redaction
- Accurate token counting via tiktoken-rs (27 models across 9 providers)
- Native language bindings (Python via PyO3, Node.js via NAPI-RS)

## Build, Test, and Lint

All commands assume the working directory is the repo root.

```bash
# Build
cargo build --release                                    # Release binary -> ./target/release/infiniloom
cargo build --workspace                                  # Debug build (all crates)

# Test
cargo test --workspace                                   # All tests
cargo test --workspace -- --nocapture                    # With stdout
cargo test -p infiniloom-engine                          # Engine crate only
cargo test proptest                                      # Property-based tests

# Lint (strict -- CI enforces this)
cargo clippy --workspace --all-targets --all-features    # Must pass with zero warnings
cargo fmt --all -- --check                               # Formatting check

# Format
cargo fmt --all                                          # Auto-format all code

# Coverage
cargo llvm-cov --workspace --all-features --html --output-dir target/coverage
```

### Makefile Shortcuts

```bash
make build          # Debug build
make build-release  # Release build
make test           # All tests (--all-features)
make lint           # Strict clippy
make fmt            # Format code
make fmt-check      # Check formatting
make ci             # Full CI pipeline: fmt-check + lint + test + coverage
make pre-commit     # Quick checks: fmt-check + check + lint
```

### CI Expectations

The GitHub Actions pipeline (`.github/workflows/ci.yml`) runs on every PR to `main`:

1. `cargo fmt --check` -- formatting must be clean
2. `cargo clippy` -- zero warnings (correctness and perf are deny-level)
3. `cargo test --workspace` -- all tests pass on Ubuntu and macOS
4. `cargo audit` -- no known vulnerabilities
5. Python and Node.js binding builds
6. Code coverage uploaded to Codecov

**Before opening a PR, always run `make pre-commit` locally.**

## Architecture

### Workspace Layout

```
infiniloom/
├── cli/                     # CLI binary (clap-based)
│   └── src/
│       ├── main.rs          # Entry point, arg parsing
│       ├── config.rs        # Config loading
│       ├── scanner.rs       # Parallel file scanning (thread-local Tree-sitter parsers)
│       └── commands/        # One file per subcommand (pack, scan, map, diff, index, etc.)
├── engine/                  # Core library (infiniloom-engine)
│   └── src/
│       ├── lib.rs           # Public API
│       ├── types.rs         # Repository, RepoFile, Symbol, SymbolKind, CompressionLevel
│       ├── newtypes.rs      # Type-safe wrappers: SymbolId, FileId, LineNumber
│       ├── error.rs         # Error types
│       ├── parser/          # Tree-sitter AST parsing (23 languages)
│       ├── tokenizer/       # Multi-model token counting (27 models)
│       ├── repomap/         # PageRank symbol ranking
│       ├── output/          # Format generators (XML, Markdown, TOON/YAML)
│       ├── chunking/        # Semantic code chunking
│       ├── embedding/       # Vector DB chunk generation (BLAKE3 content-addressable)
│       ├── document/        # Document ingestion (MD, HTML, CSV, DOCX; feature-gated)
│       ├── index/           # Symbol index, dependency graph, call graph queries
│       ├── security.rs      # Secret detection and redaction
│       ├── config.rs        # YAML/TOML/JSON config loading
│       ├── git.rs           # Git CLI wrapper (std::process::Command, not libgit2)
│       ├── ranking.rs       # File importance scoring
│       ├── budget.rs        # Token budget management
│       └── incremental.rs   # File-level caching with change detection
├── bindings/
│   ├── common/              # Shared bindings utilities
│   ├── python/              # PyO3 + Maturin
│   └── node/                # NAPI-RS
└── packages/
    └── infiniloom/          # npm CLI wrapper
```

### Data Flow

1. **Scan** -- walk directory (respecting `.gitignore`), detect languages
2. **Parse** -- Tree-sitter AST extraction for symbols (parallel, thread-local parsers)
3. **Rank** -- PageRank importance scoring across symbol graph
4. **Format** -- model-specific output (XML/Markdown/YAML)
5. **Security** -- secret detection/redaction before output

### Feature Flags (engine/Cargo.toml)

| Flag | Dependencies | Purpose |
|------|-------------|---------|
| `document` (default) | zip, quick-xml | Document ingestion (MD, HTML, CSV, DOCX) |
| `document-xlsx` | calamine | XLSX spreadsheet support |
| `watch` | notify | File watching for `pack --watch` |
| `async` | tokio | Placeholder for future async ops |
| `embeddings` | candle | Local embedding support |
| `full` | all of the above | Everything enabled |

### Linting Rules

Defined in the workspace `Cargo.toml`:

- **deny**: `clippy::correctness`, `clippy::perf`, `future_incompatible`
- **warn**: `clippy::suspicious`, `clippy::complexity`, `clippy::style`, `clippy::dbg_macro`, `clippy::todo`, `clippy::print_stdout`, `clippy::print_stderr`, `unsafe_code`
- `print_stdout`/`print_stderr` are warned everywhere except in CLI code

## Input Validation Rules

When an agent invokes `infiniloom` CLI commands or generates code that accepts external input, follow these trust boundaries:

### Trust Model

| Source | Trust Level | Validation Required |
|--------|-------------|-------------------|
| Environment variables (`INFINILOOM_*`) | Trusted | None -- set by the user or CI |
| Config files (`.infiniloom.yaml`) | Trusted | Loaded by config module |
| CLI arguments | **Untrusted** | Must validate before use |
| File contents being parsed | **Untrusted** | Handled by engine (Tree-sitter, security scanner) |

### Path Validation for CLI Arguments

When writing or reviewing code that accepts file paths from CLI args (especially `-o`/`--output`):

1. **Reject absolute paths** -- output should go relative to the working directory or repo root
2. **Reject `../` traversal** -- prevent writing outside expected directories
3. **Reject control characters** -- null bytes, newlines, etc. in filenames
4. **Use `Path::components()`** -- iterate and check for `Component::ParentDir`

```rust
// Pattern: validate output path
fn validate_output_path(path: &Path) -> Result<()> {
    if path.is_absolute() {
        anyhow::bail!("absolute output paths are not allowed");
    }
    for component in path.components() {
        if matches!(component, std::path::Component::ParentDir) {
            anyhow::bail!("path traversal (..) is not allowed in output paths");
        }
    }
    let s = path.to_string_lossy();
    if s.chars().any(|c| c.is_control()) {
        anyhow::bail!("control characters not allowed in output paths");
    }
    Ok(())
}
```

### Security Scanner Integration

- `--security-check` scans and reports secrets (pack command)
- `--redact-secrets` replaces secrets with `[REDACTED]` (pack command)
- `--no-security-scan` disables scanning (embed command)
- `fail_on_secrets` is a **config file option** (`security.fail_on_secrets`), not a CLI flag
- The embed command scans and redacts by default; use `--no-security-scan` to disable

## Environment Variables

All use double underscore (`__`) as namespace separator:

| Variable | Default | Description |
|----------|---------|-------------|
| `INFINILOOM_OUTPUT__MODEL` | `claude` | Default tokenizer model |
| `INFINILOOM_OUTPUT__FORMAT` | `xml` | Default output format |
| `INFINILOOM_OUTPUT__COMPRESSION` | `balanced` | Compression level |
| `INFINILOOM_OUTPUT__TOKEN_BUDGET` | `0` (unlimited) | Token budget limit |
| `INFINILOOM_SCAN__INCLUDE_HIDDEN` | `false` | Include hidden files |
| `INFINILOOM_SCAN__RESPECT_GITIGNORE` | `true` | Honor .gitignore |
| `INFINILOOM_SECURITY__SCAN_SECRETS` | `false` | Enable secret scanning |
| `INFINILOOM_SECURITY__REDACT_SECRETS` | `true` | Redact found secrets |

## Configuration Files

- **`.infiniloom.yaml`** or **`.infiniloom.toml`** -- project-level config
- **`.infiniloomignore`** -- additional ignore patterns (same syntax as `.gitignore`)
- Run `infiniloom init` to generate a starter config

## Pull Request Guidelines

### Before Submitting

1. Run `make pre-commit` (fmt-check + check + lint)
2. Run `cargo test --workspace` and confirm all tests pass
3. If you changed engine code, run `cargo test -p infiniloom-engine`
4. If you changed feature engineering in `parser/` or `tokenizer/`, verify with `cargo test -- --nocapture` to inspect output

### PR Conventions

- **Title**: short, imperative mood (e.g., "fix: handle empty files in embed chunker")
- **Labels**: use `bug`, `feature`, `docs`, `refactor`, `ci` as appropriate
- **Tests**: every bug fix should include a regression test; new features need unit tests
- **Breaking changes**: note in PR description if public API or CLI flags changed
- **Bindings**: if you changed the engine's public API, check whether `bindings/python/src/lib.rs` and `bindings/node/src/lib.rs` need updates

### Commit Messages

Follow conventional commits: `type: description`

- `fix:` bug fixes
- `feat:` new features
- `docs:` documentation only
- `refactor:` no behavior change
- `ci:` CI/CD changes
- `test:` adding or fixing tests
- `perf:` performance improvements

## Key Design Patterns to Follow

### Thread-Local Parsers

Tree-sitter parsers are not `Send`. Each Rayon worker gets its own via `thread_local!`:

```rust
thread_local! {
    static THREAD_PARSER: RefCell<Parser> = RefCell::new(Parser::new());
}
```

Never wrap `Parser` in `Arc<Mutex<_>>`.

### Newtype Safety

Use the wrappers in `newtypes.rs` (`SymbolId`, `FileId`, `LineNumber`) instead of raw `usize`/`u32`. This prevents mixing up IDs from different domains.

### Error Handling

- Engine code returns `Result<T, infiniloom_engine::Error>` or `anyhow::Result<T>`
- CLI code uses `anyhow::Result` throughout
- `clippy::unwrap_in_result` and `clippy::panic_in_result_fn` are warned -- do not `.unwrap()` inside functions that return `Result`

### Determinism in Embedding

The embed system guarantees identical output for identical input:

- Files processed in sorted lexicographic order
- Symbols sorted by `(line, name)` within each file
- Output chunks sorted by `(file, line, id)`
- Content-addressable IDs via BLAKE3 hash
- Cross-platform normalization (Unicode NFC, line endings)

## Language Bindings

### Python (PyO3 + Maturin)

```bash
cd bindings/python
pip install maturin
maturin develop          # Dev build
maturin build --release  # Release wheel
```

### Node.js (NAPI-RS)

```bash
cd bindings/node
npm install
npm run build
```

If you modify the engine's public API (`engine/src/lib.rs` exports), update both binding crates.

## Common Gotchas

1. **`print!`/`println!` in engine code** triggers clippy warnings. Use `log::info!`, `log::debug!`, etc. instead. Only CLI code may print to stdout/stderr.
2. **Git operations** use `std::process::Command` to call the `git` CLI, not a Rust git library.
3. **The `--max-tokens` CLI default is 0** (no limit). Config files can override this.
4. **embed `--fail-on-secrets`** does NOT exist as a CLI flag. Use the config file option `security.fail_on_secrets: true`.
5. **Minimum Rust version** is 1.91 (see `rust-version` in `Cargo.toml`).

---
> Source: [Topos-Labs/infiniloom](https://github.com/Topos-Labs/infiniloom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
