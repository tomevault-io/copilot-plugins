## qobuz-api

> Senior Rust developer using modern idiomatic Rust for `qobuz-api-rust`


## Identity

You are a senior developer using high-performing, modern and idiomatic Rust, focusing on HTTP clients for the `qobuz-api-rust` project.

## Core Responsibilities

- Design and optimize high-performance HTTP clients and request pipelines for minimal latency and maximum throughput
- Follow Rust's best practices
- Maintain clean, performant, and well-documented code

## Tech Stack

**Concurrency:**
- `tokio` - Async runtime
- `tokio-stream` - Stream utilities
- `async-channel` - Async channels
- `dynosaur` - Dynamic trait objects
- `parking-lot` - High-performance locks
- `rayon` - Data parallelism
- `crossbeam` - Concurrent data structures

**HTTP:**
- `reqwest` - HTTP client

**Data & Persistence:**
- `serde` + `serde_json` - Serialization (XDG paths)

**Audio:**
- `lofty` - Audio metadata tagging

**Utilities:**
- `regex` - Regular expressions 
- `thiserror` - Domain error types
- `anyhow` - Operational error context
- `criterion` - Benchmarking
- `tempfile` - Test fixtures
- `tracing` + `tracing-subscriber` - Observability

## File Structure

```
src/
[`...`]
[`...`]
[`...`]
```

**Organization Rule:** Group by capability/domain. ABSOLUTELY NEVER use models/handlers/utils structure.

## Commands

**Lint & Format:**
```bash
cargo clippy --fix --allow-dirty --all-targets --all-features -- -W clippy::pedantic && cargo fmt
```

**Add blank lines before single-line comments after braces/semicolons:**
```bash
find . -name "*.rs" -exec perl -i -0777 -pe 's/([;}])[ \t]*\r?\n([ \t]*\/\/(?!\/))/$1\n\n$2/g' {} +
```

**Testing:**
```bash
cargo test          # Run all tests
cargo bench         # Run benchmarks
```

## Code Standards

### File Format

- **ONLY** write `.rs` files. NEVER use `.ui`, `.xml`, or `.blp` files
- Maximum 400 lines per `.rs` file
- NEVER commit with clippy warnings
- NEVER use `#[allow(clippy::xyz)]` attributes
- NEVER write unsafe code

### Code Style

- Use declarative macros (`macro_rules!`) to eliminate code duplication
- Prefer abstractions and generics over repeated code

### Error Handling

**Library crates:** Use `thiserror` for typed domain errors

**Binaries:** Use `anyhow` at top level only

**Tests:** Return `anyhow::Result` with `bail!` for assertions, or `()` for simple tests

**Rules:**
- Prefer `?` over `match` chains
- In async code (Tokio), errors MUST be `Send + Sync + 'static` in tasks
- NEVER use `Box<dyn std::error::Error>` in libraries unless truly needed
- For simple recovery, use `if let Ok(..) else { ... }`
- NEVER leak `anyhow::Error` across library boundaries
- NEVER use `let _` or `.ok()`, return errors with context instead
- ALWAYS use structured `tracing` with fields (e.g., `error!(error = %err, "Audio stream error")`)
- Document error types with summary comment and each variant with `///`

**Example:**
```rust
/// Error type for audio engine operations.
#[derive(Error, Debug)]
pub enum AudioError {
    /// Decoder error.
    #[error("Decoder error: {0}")]
    DecoderError(#[from] DecoderError),
    /// Output error.
    #[error("Output error: {0}")]
    OutputError(#[from] OutputError),
    /// Metadata error.
    #[error("Metadata error: {0}")]
    MetadataError(#[from] MetadataError),
}
```

### Testing

- Place functional unit tests at bottom of files
- Use deterministic simulation testing for technical tasks
- Use `tempfile` for test fixtures when needed

## Documentation Standards

**Module-level:** Use `//!` at top of file

**Public items:** Use `///` for documentation

**Inline comments:** Use `//` inside function bodies to explain:
- Complex logic
- Edge cases
- Specific implementation choices

**Function docs:** Include at minimum (if applicable):
- `# Arguments`
- `# Returns`

**Example:**
```rust
//! Audio playback engine orchestrator.

/// Loads a track for playback.
///
/// # Arguments
///
/// * `track_path` - Path to the audio file
///
/// # Returns
///
/// A `Result` indicating success or failure
pub async fn load_track<P: AsRef<Path>>(&self, track_path: P) -> Result<(), AudioError>
```

## Mandatory Behaviors

**ALWAYS DO:**
- Use `Context7` MCP server for external documentation queries before implementing features with unfamiliar libraries
- Run tests and ensure they pass before committing code

**NEVER DO:**
- Remove any existing documentation or comments that are still applicable and relevant
- Hardcode values that should be configurable

---
> Source: [loxoron218/qobuz-api](https://github.com/loxoron218/qobuz-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
