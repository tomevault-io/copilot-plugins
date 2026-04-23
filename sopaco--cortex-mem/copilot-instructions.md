## cortex-mem

> > Essential context for AI coding agents working on this project.

# Cortex Memory - AI Agent Context Guide

> Essential context for AI coding agents working on this project.

---

## Project Overview

Cortex Memory is a **Rust-based AI-native memory framework** providing persistent long-term memory for AI agents via:
- Three-tier memory hierarchy (L0/L1/L2)
- Virtual filesystem with `cortex://` URI scheme
- Vector-based semantic search (Qdrant)

---

## Core Concepts

### Memory Dimensions

| Dimension | Purpose | Path Pattern |
|-----------|---------|--------------|
| `session` | Conversation memories | `cortex://session/{id}/timeline/...` |
| `user` | User-specific memories | `cortex://user/{user_id}/{category}/...` |
| `agent` | Agent-specific memories | `cortex://agent/{agent_id}/{category}/...` |
| `resources` | Shared knowledge | `cortex://resources/{name}/...` |

### Three-Tier Layers

| Layer | Tokens | Purpose | File |
|-------|--------|---------|------|
| L0 | ~100 | Quick relevance filtering | `.abstract.md` |
| L1 | ~2000 | Context understanding | `.overview.md` |
| L2 | Full | Complete original | `{name}.md` |

**Search scoring**: `0.2×L0 + 0.3×L1 + 0.5×L2`

### Data Flow

```
Message → Session (timeline/*.md) → Close → LLM Extract → Classify
    → user/agent/resources → Generate L0/L1 → Vector Index (Qdrant)
```

---

## Workspace Structure

```
cortex-mem-core/      # Core business logic (src/lib.rs)
cortex-mem-service/   # REST API server (src/main.rs)
cortex-mem-cli/       # CLI tool (src/main.rs)
cortex-mem-mcp/       # MCP protocol server (src/main.rs)
cortex-mem-tools/     # MCP tool definitions (src/lib.rs)
cortex-mem-config/    # Configuration parsing (src/lib.rs)
cortex-mem-rig/       # Rig integration (src/lib.rs)
cortex-mem-insights/  # Web dashboard (Svelte)
```

---

## Build & Test

```bash
cargo build --workspace
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --check

# Run service
cargo run --bin cortex-mem-service -- --config config.toml

# Run CLI
cargo run --bin cortex-mem -- --help
```

---

## Coding Conventions

### Rust Style

- **Async by default**: Use `async fn` for I/O operations
- **Arc for sharing**: `Arc<T>` for shared state across threads
- **RwLock for state**: `RwLock<T>` for mutable shared state
- **Result handling**: `anyhow::Result` for app code, `thiserror` for library errors

```rust
// Good: Use tokio::fs, not std::fs in async context
pub async fn read_file(&self, path: &str) -> Result<String> {
    tokio::fs::read_to_string(path).await?;
}

// Avoid: Blocking in async
pub async fn bad(&self) -> Result<String> {
    std::fs::read_to_string("file.txt")?; // Blocks!
}
```

### Error Handling

```rust
// Define errors in error.rs with thiserror
#[derive(Debug, thiserror::Error)]
pub enum Error {
    #[error("Invalid URI: {0}")]
    InvalidUri(String),
}

// Use anyhow in handlers and CLI
pub fn do_work() -> anyhow::Result<()> { ... }
```

### API Design

- Avoid bool or ambiguous `Option` parameters that create unclear call sites
- Prefer enums, builder patterns, or named methods

```rust
// Bad: foo(false, None, true) - what do these mean?
// Good: foo(FooOptions { verbose: false, output: None, force: true })
// Good: foo.quiet().without_output().force()
```

### Module Size

- Target modules under 500 LoC (excluding tests)
- If a file exceeds 800 LoC, add new functionality in a new module
- Extract related tests when moving code to new modules

### Documentation

- Add doc comments to public traits explaining their role
- When adding/changing APIs, update relevant docs in `.ai-context/` if applicable

---

## Key Files

| What | Where |
|------|-------|
| Core types | `cortex-mem-core/src/types.rs` |
| URI parsing | `cortex-mem-core/src/filesystem/uri.rs` |
| Search engine | `cortex-mem-core/src/search/vector_engine.rs` |
| Session manager | `cortex-mem-core/src/session/manager.rs` |
| LLM client | `cortex-mem-core/src/llm/client.rs` |
| API handlers | `cortex-mem-service/src/handlers/` |
| MCP tools | `cortex-mem-tools/src/tools/` |
| Config schema | `cortex-mem-config/src/lib.rs` |

---

## Configuration

Via `config.toml`. Key environment variables:

```bash
OPENAI_API_KEY      # or LLM_API_KEY
EMBEDDING_API_KEY   # Optional, defaults to LLM key
QDRANT_URL          # Optional, defaults to localhost:6334
```

---

## Multi-Tenancy

```
{data_dir}/tenants/{tenant_id}/
├── session/
├── user/
├── agent/
└── resources/

Qdrant collection: "{base}_{tenant_id}"
```

Switch tenant via `POST /api/v2/tenants/switch`. Complete isolation between tenants.

---

## Common Tasks

### Adding a New API Endpoint

1. Add handler in `cortex-mem-service/src/handlers/`
2. Register route in `cortex-mem-service/src/routes/mod.rs`
3. Add models in `cortex-mem-service/src/models.rs` if needed

### Adding a New MCP Tool

1. Define tool schema in `cortex-mem-tools/src/tools/`
2. Add operation in `cortex-mem-tools/src/operations.rs`
3. Register in `cortex-mem-mcp/src/service.rs`

### Adding a New Core Module

1. Create module directory in `cortex-mem-core/src/`
2. Add `pub mod name;` to `lib.rs`
3. Re-export key types in `lib.rs` if public API
4. Update `.ai-context/` if it's a major architectural addition

---

## Documentation Maintenance

### When to Update

| Trigger | Action |
|---------|--------|
| New crate | Update Workspace Structure |
| New core module (>200 LoC) | Update Key Files |
| Architecture decision | Add to `.ai-context/decisions.md` |
| New config field (non-obvious) | Note in Configuration |

### When NOT to Update

- New functions/methods
- New API endpoints
- Bug fixes
- Internal refactoring

### `.ai-context/` Files

| File | Purpose |
|------|---------|
| `decisions.md` | Architecture Decision Records |
| `uri-structure.md` | Complete URI scheme reference |

---

## Notes

- Default `user_id` is `"default"` in URI paths
- L0/L1 layer files (`.abstract.md`, `.overview.md`) are hidden
- Session extraction only triggers on explicit `close` call
- Use `include_layers=true` in `cortex_ls` to see layer files

---
> Source: [sopaco/cortex-mem](https://github.com/sopaco/cortex-mem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
