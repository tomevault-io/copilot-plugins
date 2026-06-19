## rust-forge-skill

> This skill makes AI agents generate idiomatic, maintainable, secure, and testable Rust projects. It provides templates, guides, validation scripts, checklists, and enforcement rules that agents follow strictly when producing Rust code.

# Rust Forge Skill

## Purpose

This skill makes AI agents generate idiomatic, maintainable, secure, and testable Rust projects. It provides templates, guides, validation scripts, checklists, and enforcement rules that agents follow strictly when producing Rust code.

Agents using this skill will:
- Scaffold projects that pass `cargo clippy -D warnings` and `cargo test`
- Apply correct error handling strategies (`thiserror` for libraries, `anyhow` for applications)
- Enforce memory safety via SAFETY documentation on all unsafe blocks
- Use appropriate layer architecture (API → Service → Repository → Domain)
- Configure CI/CD with MSRV enforcement, security audits, and coverage tracking
- Avoid the 25+ anti-patterns catalogued in this document

---

## When To Use

Use this skill when an agent needs to:

| Scenario | Template |
|---|---|
| New Rust CLI application | `cli-app/` |
| New Rust HTTP API / web service | `axum-api/` |
| Publishable library crate | `library-crate/` |
| Cargo workspace with multiple crates | `workspace-service/` + `scripts/generate_cargo_workspace.sh` |
| Rust FFI wrapper for C/C++ library | `ffi-wrapper/` |
| Rust WASM module for browser or edge runtime | `wasm-module/` |
| Refactor low-quality Rust to idiomatic patterns | `guides/12-rust-anti-patterns.md` + `guides/03-error-handling-anyhow-thiserror.md` |
| Add tests, benchmarks, or fuzz targets | `guides/06-testing-bench-fuzz.md` |
| Add CI/CD pipeline | `ci/github-actions-rust.yml` + `ci/github-actions-rust-security.yml` |
| Add security gates (audit, unsafe review) | `guides/08-security-unsafe-audit.md` + `scripts/audit_unsafe.sh` |
| Audit existing unsafe code | `examples/agent-prompts/unsafe-audit.md` + `scripts/audit_unsafe.sh` |

**Do not use** this skill for: fixing compilation errors in existing code (use `systematic-debugging`), researching Rust internals (use web search), or writing tutorial content.

---

## Self-Testing

The skill pack includes self-test scenarios in `self-tests/` that verify an agent can follow the skill guidance to complete real tasks. Each scenario gives an agent a concrete task plus embedded example code, and validates the agent's output against an expected report shape. See `self-tests/README.md` for execution instructions.

---

## MCP/Agent Integration Layer

This skill includes a machine-readable MCP layer for programmatic loading by AI agents and tool runners.

| File | Purpose |
|---|---|
| `mcp/manifest.example.json` | Machine-readable skill manifest (name, version, capabilities, validation commands) |
| `mcp/tool-contract.md` | Formal agent protocol contract (input schema, output schema, safety invariants) |
| `mcp/agent-loader-instructions.md` | Step-by-step instructions for agents that need explicit guidance |
| `examples/agent-prompts/load-rust-forge-skill.md` | Copy-paste prompt for any agent to load this skill |

### Loading Methods

**Claude Code:** The `Skill` tool reads `SKILL.md` automatically. Use `/skill rust-forge-skill` to explicitly invoke.

**Other agents and tool runners:** Use `mcp/manifest.example.json` for auto-discovery, `mcp/tool-contract.md` for protocol compliance, and `mcp/agent-loader-instructions.md` for step-by-step guidance. See `mcp/README.md` for full details on all supported loading methods (IDE plugins, MCP servers, repo-scanning agents, CLI agents).

### Capability Routing

| Capability ID | Use Case | Required Inputs |
|---|---|---|
| `rust_project_scaffold` | New project from template | `project_path`, `project_type`, `project_name` |
| `rust_project_audit` | Audit existing project | `project_path` |
| `rust_error_refactor` | Refactor error handling | `project_path`, `project_type` |
| `rust_async_service` | Async API with Axum | `project_path`, `project_type` |
| `rust_ffi_audit` | Unsafe/FFI audit | `project_path` |
| `rust_security_validation` | Security gate run | `project_path` |

See `mcp/manifest.example.json` for the full capability definitions including optional inputs and expected outputs.

### Safety Invariants (Non-Negotiable)

All agents must uphold these invariants regardless of task:

1. **No destructive rewrite without explicit user request** — do not delete files outside `project_path`
2. **No unsafe without audit** — every `unsafe` block needs a `SAFETY` comment
3. **No dependency bloat** — confirm necessity before adding deps
4. **No secret exposure** — no API keys or tokens in source, logs, or output
5. **No unwitnessed test passing** — run and report actual validation output
6. **No unvalidated deploy** — verdict must match actual validation results

See `mcp/tool-contract.md` for the full safety invariant definitions.

---

## Agent Operating Rules

Agents **MUST** follow every rule in this section. Violations require written justification.

### Project Inspection Before Changes

```
Before adding a crate, dependency, or module:
1. Read existing Cargo.toml — understand current deps, edition, workspace membership
2. Read existing src/lib.rs or src/main.rs — understand current architecture
3. Check for .clippy.toml and rustfmt.toml — do not override without reason
4. Check for existing workspace membership — do not add crates outside the workspace
```

### Async Rules

- Do not introduce `async` unless the workload is I/O-bound and concurrent or the framework requires it (axum, sqlx, reqwest).
- For CLI tools that do simple sequential work: use synchronous Rust.
- For long-running services: use `tokio` with `#[tokio::main]` and `tokio::select!` for shutdown.
- Never use `async` in library traits without `#[async_trait]`.

### Unsafe Rules

- Do not use `unsafe` unless interfacing with C/FWI, low-level memory operations, or performance-critical code that has been profiled.
- Every `unsafe` block **must** have a `SAFETY` comment covering: (1) what invariant must hold, (2) why it holds at this call site, (3) what UB occurs if violated.
- Keep unsafe surface area minimal. Unsafe blocks must be wrapped in safe public APIs immediately.
- Never use `unsafe` in a public library API unless unavoidable — prefer safe wrappers.

### Error Handling Rules

- Do not use `.unwrap()` or `.expect()` in any production path (library code, API handlers, service layer).
- `.unwrap()` is **tolerated only in**: tests, example code, and temporary prototyping.
- Do not use `Box<dyn Error>` as a return type — use typed error enums.
- Do not hide errors behind `String` — use `thiserror` enums with structured variants.
- Library crates **must not** use `anyhow`. Use `thiserror` for domain errors.
- Application/CLI boundary code **may** use `anyhow` for ergonomic error propagation.
- Never swallow errors silently. Every `Result` must be handled via `?`, `match`, or explicit `unwrap_or_else` with logging.

### Code Organization Rules

- Do not create `main.rs` files larger than 100 lines. Extract logic to `lib.rs` or a `commands/` module.
- Do not mix CLI parsing (`clap`), business logic, I/O, and domain models in a single file.
- Follow layer architecture: `api/` → `service/` → `repository/` → `domain/`.
- Domain layer has **zero** external runtime dependencies (no `tokio`, `sqlx`, `axum`).
- Repository layer defines **traits** (in domain), implements them in `infrastructure/`.

### Dependency Rules

- Do not add dependencies casually. Every dependency is a maintenance burden.
- Before adding a dependency: confirm it's actively maintained, has no security advisories, and the same functionality cannot be achieved with fewer deps.
- Prefer crates with explicit feature flags over all-features-on defaults.
- Use `cargo outdated` to check for dependency updates quarterly.
- Use `cargo deny` in CI to block banned crates, unacceptable licenses, and security advisories.

### Testing Rules

- Write tests **next to the behavior** they verify (in the same file, `#[cfg(test)]` module).
- Cover the error path, not just the happy path.
- Use `proptest` for input validation functions, `quickcheck` for property testing.
- Use `mockall` for unit test mocking of traits, `wiremock` for HTTP client testing.
- Never use `.unwrap()` in test utility functions that are called by production code paths.

### Formatting and Linting Rules

- Always run `cargo fmt` after every change. Never skip it.
- Always run `cargo clippy --workspace --all-targets --all-features -- -D warnings` before reporting success.
- Never bypass clippy warnings with `#[allow(...)]` unless the linter is provably wrong. Document why.
- Never commit with `TODO`, `FIXME`, or debug `println!` statements in production code.

---

## Rust Project Decision Tree

```
Is the output a published library (consumed by other Rust projects)?
  YES → library-crate template
        Architecture: domain/service/error layers, zero external deps in domain
        Error: thiserror
        Testing: proptest, unit tests
        CI: MSRV check, cargo doc
        Guides: 01, 02, 03, 06, 10

  NO → Is it a CLI tool with command-line argument parsing?
    YES → cli-app template
          Architecture: commands/ module, clap derive, tracing
          Error: anyhow (boundary only)
          Testing: integration tests
          CI: standard CI
          Guides: 05, 03, 01

    NO → Does it serve HTTP/REST/GraphQL requests?
      YES → axum-api template
            Architecture: api/handlers → service/ → repository/ → domain/
            Error: thiserror in domain, anyhow at boundary
            Async: tokio + axum + tower-http
            Testing: tower mock, wiremock
            CI: standard CI + doc
            Guides: 04, 01, 03, 02, 06

      NO → Does it interface with C/C++ via FFI?
        YES → ffi-wrapper template
              Architecture: bindings.rs (generated) → wrapper.rs (safe) → error.rs
              Unsafe: mandatory SAFETY comments, Send+Sync impl
              Error: thiserror
              Testing: integration tests against C library
              CI: audit_unsafe.sh mandatory
              Guides: 07, 08

        NO → Is the target WASM (browser or edge runtime)?
          YES → wasm-module template
                Architecture: lib.rs with wasm-bindgen exports, utils.rs for pure Rust
                Unsafe: minimal (WASM is sandboxed)
                Testing: wasm-bindgen-test
                CI: wasm-pack build verification
                Guides: 09

          NO → Does it require multiple related crates (shared domain, API, client SDK)?
            YES → workspace-service template
                  Architecture: workspace with core/, api/, client/ crates
                  Each crate follows its own layer rules
                  CI: cargo check --workspace
                  Scripts: generate_cargo_workspace.sh
                  Guides: 02, 04, 01, 10

            NO → Default to cli-app template as fallback
```

---

## Default Tooling

All projects generated by this skill assume the following tools are available. Install via `cargo install <tool>` or via system package manager.

| Tool | Purpose | When To Use |
|---|---|---|
| `cargo fmt` | Format code | Every change, every commit |
| `cargo clippy` | Lint | Every change, CI gate |
| `cargo test` | Unit + integration tests | Every change, CI gate |
| `cargo nextest` | Faster parallel test runner | Replace `cargo test` in CI when available |
| `cargo audit` | Security vulnerability scanner | CI gate, before release |
| `cargo deny` | License + advisory checker | CI gate |
| `cargo outdated` | Dependency update checker | Monthly review |
| `cargo machete` | Dead code detection | Before releases |
| `cargo llvm-cov` | Code coverage | When coverage tracking is required |
| `criterion` | Benchmarks | Performance regression testing |
| `proptest` | Property-based testing | Input validation functions |
| `quickcheck` | Property-based testing | Alternative to proptest |
| `cargo fuzz` | Fuzz testing | Parsers, input-handling code |
| `wasm-pack` | WASM build + test | WASM modules only |
| `bindgen` | C header → Rust bindings | FFI projects |
| `cargo-diet` | Binary size reduction | Before release |

---

## Default Rust Standards

### Edition and MSRV

- **Default edition: Rust 2024**
- **Default MSRV: 1.85.0**
- Libraries **must** declare MSRV in `Cargo.toml` `[package]` as `rust-version = "1.85"`
- CI runs `check_msrv.sh` against the declared MSRV
- Override per-project: `RUST_MSRV=1.82.0 ./scripts/check_msrv.sh`

### Workspace Dependency Management

All workspace dependencies are defined **once** in the workspace root `Cargo.toml` under `[workspace.dependencies]` and referenced in crate `Cargo.toml` files as `tokio.workspace = true`.

```toml
# Workspace root
[workspace.dependencies]
tokio = { version = "1.40", features = ["full"] }
thiserror = "2.0"
anyhow = "1.0"
tracing = "0.1"

# Crate Cargo.toml
[dependencies]
tokio.workspace = true
thiserror.workspace = true
```

### Feature Flags

Every crate uses explicit feature flags. No all-features-on defaults.

```toml
[features]
default = []
sqlite = ["sqlx/sqlite", "tokio/fs"]
postgres = ["sqlx/postgres"]
serde = ["dep:serde"]  # Enable only when needed
```

### Logging

- **Non-trivial applications**: `tracing` with structured fields. Never `println!` / `eprintln!` in production.
- **Simple CLI tools (< 500 lines)**: `log` + `env_logger` acceptable.
- Log levels controlled via `RUST_LOG` environment variable.

### Error Handling

| Layer | Strategy |
|---|---|
| Library/domain | `thiserror` — typed error enums with structured variants |
| Application boundary (CLI, API handler) | `anyhow` — ergonomic `?` propagation, `.context()` |
| FFI boundary | `thiserror` wrapping C error codes |

**Never** use `anyhow` in library crates. **Never** use `thiserror` at CLI entry points where any error can occur.

### Security Review

Every project (especially FFI) requires:
- `cargo audit` in CI
- `cargo deny check advisories licenses` in CI
- `scripts/audit_unsafe.sh` run manually before delivery
- No secrets in source code (use env vars, `.env.example` documents format)
- Input validation on all external boundaries

### Documentation

- All public items have doc comments (`///`).
- Libraries build `cargo doc --workspace --all-features --no-deps` without warnings.
- README.md documents: build commands, env vars, architecture, quality gates.
- `.env.example` lists all environment variables with descriptions.

### CI Requirements

Every Rust project CI **must** include:

```yaml
jobs:
  fmt:        cargo fmt --all -- --check
  clippy:     cargo clippy --workspace --all-targets --all-features -- -D warnings
  test:       cargo test --workspace --all-features
  doc:        cargo doc --workspace --all-features --no-deps  # libraries only
  msrv:       cargo check --workspace --all-features  # MSRV toolchain
  audit:      cargo audit
  deny:      cargo deny check advisories licenses
```

---

## Validation Commands

### Minimal Validation (per-change)

```bash
cargo fmt --all
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo test --workspace --all-features
```

### Full Validation (before PR/merge)

```bash
# Phase 1: Format + Lint
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings

# Phase 2: Tests
cargo test --workspace --all-features
cargo test --workspace --doc  # doc tests

# Phase 3: Documentation (libraries)
cargo doc --workspace --all-features --no-deps

# Phase 4: MSRV (libraries)
./scripts/check_msrv.sh

# Phase 5: Security
cargo audit
cargo deny check advisories licenses ban
./scripts/audit_unsafe.sh
```

### Security Validation (CI + manual)

```bash
cargo audit
cargo deny check advisories licenses
# Unsafe audit for FFI projects
./scripts/audit_unsafe.sh
# Check for outdated deps
cargo outdated --workspace || true
# Dead code
cargo machete --strict || true
```

### Release Validation

```bash
cargo build --release
cargo test --workspace --all-features
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
# Verify binary
./target/release/{{crate_name}} --version
```

---

## Output Expectations For Agents

When an agent modifies or generates a Rust project using this skill, the final report **must** include:

### Files Changed

```
| File | Change |
|------|--------|
| Cargo.toml | Added serde dependency, updated version |
| src/domain/models.rs | Added User struct with validation |
| src/error.rs | Added NotFound variant to Error enum |
```

### Architectural Decisions

```
- Chose repository trait pattern (guides/01) to decouple service from DB implementation
- Used thiserror instead of anyhow because this is a library consumed by other crates
- Separated domain from infrastructure: domain/ has zero tokio/sqlx deps
```

### Dependencies Added

```
| Dependency | Version | Reason |
|---|---|---|
| thiserror | 2.0 | Typed domain errors, avoids String-based errors |
| uuid | 1.10 | User ID generation, v4 for random IDs |
```

### Validation Commands Run

```
cargo fmt --all -- --check    # PASS
cargo clippy --workspace --all-targets --all-features -- -D warnings    # PASS (0 warnings)
cargo test --workspace --all-features    # PASS (12/12 tests)
cargo doc --workspace --all-features --no-deps    # PASS (no warnings)
```

### Known Limitations

```
- MSRV bumped from 1.82 to 1.85 due to thiserror 2.0 requirement
- Connection pool settings (min=2, max=10) are tuned for development; production should tune
```

### Next Recommended Steps

```
1. Add integration tests for repository layer using wiremock
2. Configure cargo-deny in CI to block yanked dependencies
3. Add database migration tooling (sqlx-cli or refinery)
```

---

## Anti-Patterns

The following table lists **forbidden** and **discouraged** patterns. Agents must not generate these in production code without explicit written justification.

### Ownership and Memory

| Bad | Prefer | Reason |
|---|---|---|
| `clone()` in hot paths without profiling | Borrow via `&` / `&mut` | `clone()` allocates; hot paths need zero-cost abstractions |
| `Box<Vec<T>>` instead of `Vec<T>` on stack | `Vec<T>` directly | Unnecessary indirection, worse cache behavior |
| `static mut` for global state | `static` with `Mutex` or `OnceLock` | `static mut` is UB under Rust's memory model |
| `Rc<RefCell<T>>` for shared mutation | `Arc<Mutex<T>>` or actor pattern | `Rc<RefCell>` is not `Send`/`Sync`, breaks async |
| `unwrap()` on `Vec::first()` / `get()` | `ok()` or `and_then()` | Returns `None` on failure, not panic |

### Lifetimes

| Bad | Prefer | Reason |
|---|---|---|
| Elided lifetimes on function signatures that need explicitness | Name all lifetimes explicitly | Hidden borrow relationships cause confusing errors later |
| `'static` bound when not required | Inferred or specific lifetime | Unnecessarily restricts caller types |
| `impl Trait` return type when `Box<dyn Trait>` is correct | `Box<dyn Trait>` for type erasure | `impl Trait` in return position is for generic implementations |

### Errors

| Bad | Prefer | Reason |
|---|---|---|
| `anyhow::Error` in library code | `thiserror` enum | Loses type information, defeats error categorization |
| `String` as error type | Typed error enum | `String` loses structure, cannot be matched exhaustively |
| `.unwrap()` to "just get the value" | `?` + typed error | Panics on None/Err; hides failure modes |
| `.expect("...")` in library code | Same as `.unwrap()` — forbidden | Same reason as `.unwrap()` |
| `error.to_string()` in error context | `error.to_string()` → `anyhow!(...)` | Loses error chain context |
| Panic for "impossible" cases | `unreachable!()` with comment | `unreachable!()` is faster and self-documents intent |

### Async

| Bad | Prefer | Reason |
|---|---|---|
| Blocking sync I/O in async fn (`std::fs::read`) | `tokio::fs` or `tokio::task::spawn_blocking` | Blocks the async executor thread |
| `block_on` inside an async context | Restructure using `.await` | Deadlock risk, defeats async composition |
| `async` in library trait without `#[async_trait]` | `#[async_trait]` on trait definition | Object-safe async traits require `#[async_trait]` |
| `tokio::spawn` without awaiting the handle | Store handle and `.await` or `.abort()` | Spawned tasks may outlive their intended scope |
| `select!` without `biased` and without random branch order | Document branch priority, consider `biased` | fairness != correctness; check priority |

### Traits and Generics

| Bad | Prefer | Reason |
|---|---|---|
| Generic bounds scattered across impl blocks | Consolidate bounds in trait definition | Improves readability, enables better error messages |
| `impl Trait` without understanding its limitations | Concrete type or `Box<dyn Trait>` | `impl Trait` is not the same as generics in a trait |
| Blanket impl without clear coherence plan | Explicit impls per type | Orphan rules prevent downstream implementations |

### Module Organization

| Bad | Prefer | Reason |
|---|---|---|
| Giant `main.rs` (300+ lines) | Extract to `lib.rs` + `commands/` module | Violates single responsibility, hard to test |
| `mod utils;` full of unrelated helpers | Split into named modules by concern | `utils` is a code smell indicating missing domain types |
| Circular module dependencies | Split into separate crates or use traits | Rust module system does not support cycles |
| Exposing internal types via `pub use` without documentation | `pub(crate)` or documented re-exports | Public API surface is a contract |

### Unsafe Code

| Bad | Prefer | Reason |
|---|---|---|
| `unsafe { ... }` without `SAFETY` comment | Every unsafe block needs `SAFETY: invariant / hold / UB` | Unsafe is a promise about invariants; undocumented = unverifiable |
| Raw `*mut T` held beyond a single function | Wrap immediately in a safe type with `Drop` | Raw pointers leak memory safety through the API |
| `unsafe impl` without safety documentation | `// SAFETY: ...` before every `unsafe impl` | `unsafe impl` asserts trait safety requirements |
| `unsafe` block longer than 10 lines | Refactor into a separate safe wrapper | Long unsafe blocks are hard to verify; split for auditability |
| `std::mem::zeroed()` on non-`Copy` types | `MaybeUninit<T>` + explicit initialization | Zeroed memory is not a valid value for most types |

### FFI

| Bad | Prefer | Reason |
|---|---|---|
| `*const T` when meaning `*mut T` | Use correct mutability, document ownership | C ABI is explicit about mutability; mismatches cause UB |
| `libc::c_char` interpreted as `u8` without validation | Validate or use `CStr` / `CString` | UTF-8 invalidity in C strings causes silent corruption |
| Calling C functions without checking return codes | Map C errors to Rust error enum | Silent failure hides errors from callers |
| Pointer returned from C not validated before use | Check `null()` before dereferencing | Null pointer dereference is UB |

### Tests

| Bad | Prefer | Reason |
|---|---|---|
| `unwrap()` in test helper called by production paths | Return `Result` and propagate | Helper's panic is cryptic when called from production |
| Tests only covering happy path | Test error variants explicitly | Untested error paths break in production |
| `#[ignore]` without a tracking issue or reason | Fix the flaky test or remove | Ignored tests rot and provide false confidence |
| Mocking without `mockall` | `mockall` with `#[mockall::automock]` | Manual mocks drift from real implementations |

### Dependencies

| Bad | Prefer | Reason |
|---|---|---|
| `features = ["full"]` all-features-on default | Explicit features per use | Bloats binary, slows compile, may introduce unwanted behavior |
| Adding a crate for one trivial function | Implement inline or use std | Every dep is a maintenance and security burden |
| `lazy_static` / `once_cell` without considering `std` | Use `std::sync::OnceLock` | `OnceLock` is stable and zero-dependency |
| `log` + `env_logger` in async servers | `tracing` with `tracing-subscriber` | `tracing` has structured fields, async-native |

### Workspaces

| Bad | Prefer | Reason |
|---|---|---|
| `**/*.rs` glob imports for modules | Explicit `mod` statements | Glob imports obscure the module graph |
| Crates sharing `unwrap()` via dev-dependencies | Add a `test-util` feature flag | Prod builds should not pay for test utilities |
| Different MSRV per crate without documented justification | Workspace MSRV is the floor | Crates with lower MSRV cannot depend on higher-MSRV crates |
| Publishing workspace crates individually without lockstep versioning | Use `cargo workspaces version bump --all` | API consistency across workspace crates |

### Performance

| Bad | Prefer | Reason |
|---|---|---|
| `String` where `&str` suffices | `&str` as function parameter | Avoids allocation in hot paths |
| `format!()` in logging hot paths | Use `tracing` with `tracing::info!(field = value, ...)` | `format!()` allocation dominates log overhead |
| `collect::<Vec<_>>()` without size hint | `.with_capacity()` + loop or `.size_hint()` | Avoids reallocation |
| Premature SIMD without profiling | Profile first, confirm bottleneck | SIMD adds complexity; verify it's the bottleneck |

### Formatting and Style

| Bad | Prefer | Reason |
|---|---|---|
| Comments explaining **what** code does | Code is self-documenting; explain **why** | Comments that restate code rot and mislead |
| `// TODO: refactor later` without issue reference | Link to issue tracker | TODOs without owners become permanent debt |
| Magic numbers without `const` | Named constants (`const MAX_RETRIES: u32 = 3`) | Magic numbers obscure intent and resist changes |

---

## Final Agent Checklist

Run this checklist before finishing any Rust project work:

```
PRE-COMMIT GATE (all must pass):
[ ] cargo fmt --all -- --check    # No format deviations
[ ] cargo clippy --workspace --all-targets --all-features -- -D warnings    # Zero warnings
[ ] cargo test --workspace --all-features    # All tests pass

ARCHITECTURE:
[ ] No .unwrap() / .expect() in production code paths
[ ] No unsafe without SAFETY comment
[ ] No println! / eprintln! in production code
[ ] Layer dependencies flow inward (api → service → domain)
[ ] Domain layer has zero tokio / sqlx / axum dependencies

DEPENDENCIES:
[ ] No casual dependency additions
[ ] All dependencies use explicit feature flags
[ ] No static mut

ERROR HANDLING:
[ ] Library code uses thiserror (not anyhow)
[ ] Application boundary uses anyhow (not thiserror at entry point)
[ ] No String-based error hiding

DOCUMENTATION:
[ ] All public items have doc comments
[ ] README.md is current (build commands, env vars, architecture)
[ ] .env.example lists all required environment variables

TESTS:
[ ] Tests next to behavior they verify
[ ] Error paths are tested, not just happy path
[ ] Test utilities do not use unwrap() in production-callable functions

REPORTING:
[ ] Files changed documented
[ ] Dependencies added documented with reason
[ ] Validation output included
[ ] Known limitations documented
[ ] Next steps documented
```

---

## Quick Reference

**Template selection:** CLI → `cli-app/` | API → `axum-api/` | Library → `library-crate/` | FFI → `ffi-wrapper/` | WASM → `wasm-module/` | Workspace → `workspace-service/`

**MSRV:** 1.85.0 (Rust 2024). Libraries declare in `[package] rust-version`.

**Quality gate:** `cargo fmt --all -- --check && cargo clippy --workspace --all-targets --all-features -- -D warnings && cargo test --workspace --all-features`

**Anti-patterns summary:** No `.unwrap()` in prod | No unsafe without SAFETY | No `anyhow` in libs | No `println!` in prod | No `static mut` | No feature creep

**Guides quick-ref:**
- Error handling: `guides/03-error-handling-anyhow-thiserror.md`
- Async: `guides/04-async-tokio-axum.md`
- FFI: `guides/07-ffi-c-cpp.md`
- Security: `guides/08-security-unsafe-audit.md`
- Anti-patterns: `guides/12-rust-anti-patterns.md`
- Testing: `guides/06-testing-bench-fuzz.md`

---
> Source: [Arvuno/rust-forge-skill](https://github.com/Arvuno/rust-forge-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
