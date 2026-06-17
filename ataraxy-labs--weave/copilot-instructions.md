## weave

> Entity-level semantic merge driver for Git. Resolves conflicts at the function/class level instead of lines.

# weave

Entity-level semantic merge driver for Git. Resolves conflicts at the function/class level instead of lines.

## Structure

Cargo workspace at repo root:
- `weave-core` — merge engine, entity matching, inner merge
- `weave-crdt` — CRDT coordination layer
- `weave-driver` — Git merge driver binary (called by git)
- `weave-cli` — CLI (`weave setup`, `weave bench`, `weave preview`)
- `weave-mcp` — MCP server (15 tools for agent coordination)
- `weave-github` — GitHub integration

## Build & Test

```bash
cargo build --release                # all binaries
cargo test --workspace               # 124 tests
```

Binaries: `target/release/weave`, `target/release/weave-driver`, `target/release/weave-mcp`

## Key Paths

- Merge logic: `crates/weave-core/src/`
- CRDT ops: `crates/weave-crdt/src/`
- Driver entry: `crates/weave-driver/src/main.rs`
- CLI commands: `crates/weave-cli/src/`
- MCP tools: `crates/weave-mcp/src/`
- Integration tests: `crates/weave-core/tests/integration.rs`

## How Merging Works

1. sem-core extracts entities from base, ours, theirs
2. Match entities by name across versions
3. For each entity: unchanged, added, deleted, or modified
4. Modified entities get inner-merged (chunk by indentation, match by name)
5. Only true conflicts produce conflict markers

## Conventions

- Depends on `sem-core` for entity extraction
- sem-core extracts classes AND methods as separate entities (methods have `parentId`)
- Inner merge chunks by indentation, matches by name
- Benchmark: 31/31 clean merges (100%) vs git 15/31 (48%)
- `vendored-openssl` feature for cross-compilation
- Release on tag push (`v*`)
- Homebrew: `brew install ataraxy-labs/tap/weave`
- License: MIT OR Apache-2.0

---
> Source: [Ataraxy-Labs/weave](https://github.com/Ataraxy-Labs/weave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
