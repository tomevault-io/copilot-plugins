## jar

> - `spec/` — JAR formal specification (Lean 4)

@spec/AGENTS.md
@grey/AGENTS.md

## Monorepo Layout

- `spec/` — JAR formal specification (Lean 4)
- `grey/` — Grey protocol node (Rust)
- `spec/tests/vectors/` — Shared conformance test vectors (used by both)

## Test Vectors

Shared conformance test vectors live in `spec/tests/vectors/<subsystem>/`. Three variants: `gp072_tiny`, `gp072_full`, `jar1`. Grey only tests `jar1`.

### Rebuild PVM blobs for test vectors

The accumulate test vectors reference compiled PVM service blobs (`spec/tests/vectors/accumulate/blobs/`). When service source changes (e.g., host call numbers, REPLY convention), rebuild:

```bash
cd grey
cargo clean -p spec-tests && cargo build --release -p spec-tests
cargo run --release -p spec-tests -- bless ../spec/tests/vectors/accumulate/blobs
```

### Rebless jar1 test vectors

After changing the Lean spec or rebuilding blobs, regenerate expected outputs:

```bash
cd spec
lake build jarstf
.lake/build/bin/jarstf --variant jar1 --bless accumulate tests/vectors/accumulate
```

To bless a specific subsystem for a specific variant:
```bash
.lake/build/bin/jarstf --variant <variant> --bless <subsystem> tests/vectors/<subsystem>
```

**IMPORTANT**: `gp072_tiny` and `gp072_full` test vectors must remain unchanged unless the GP spec changes. Only rebless `jar1` for JAR-specific changes.

### Verify all variants pass

```bash
cd spec && make test                           # Lean spec (all 3 variants)
cd grey && cargo test -p grey-state            # Rust STF (jar1 only)
```

## Conventions

- Commit early, commit often. Small logical changes per commit.
- Don't "work around" an issue. Always fix the root cause.
- Strict interfaces: require all fields, fail early, be loud about failures. Never silently default missing input — if a field is expected, error when it's absent. Fix callers, not callees.
- Run `cargo fmt --all` and `cargo clippy --workspace --all-targets -- -D warnings` before submitting a PR. CI enforces both.

---
> Source: [jarchain/jar](https://github.com/jarchain/jar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
