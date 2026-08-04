## testing-grounds

> This document provides essential context for AI assistants working on this codebase. Always review this before making changes.

# AI Assistant Guidelines for Tauri Rust WASM Project

This document provides essential context for AI assistants working on this codebase. Always review this before making changes.

## Project Architecture Principles

### 1. Modular Design (800-Line Rule)
- **CRITICAL**: No single file should exceed 800 lines
- Split large modules into sub-modules when approaching 600 lines
- Use trait objects and interfaces to maintain loose coupling
- Example structure for large features:
  ```
  src/modules/feature/
  ├── mod.rs          # Public API (< 100 lines)
  ├── types.rs        # Type definitions (< 300 lines)
  ├── impl.rs         # Core implementation (< 400 lines)
  ├── handlers.rs     # Event handlers (< 400 lines)
  └── tests.rs        # Module tests
  ```

### 2. Layer Separation
Always maintain clear boundaries between:
- **Core Logic** (src/core/): Pure Rust, no platform dependencies
- **WASM Layer** (src/wasm/): WebAssembly bindings and browser APIs
- **Tauri Layer** (src/tauri/): Desktop-specific functionality
- **Modules** (src/modules/): Feature-specific, self-contained units

## System Dynamics to Check

### Before Any Code Changes

1. **Docker Environment Status**
   ```bash
   docker-compose ps  # Ensure container is running
   docker-compose logs tauri-dev  # Check for errors
   ```

2. **Dependency Versions**
   - Check Cargo.toml for version conflicts
   - Verify wasm-bindgen versions match between Rust and JS
   - Ensure Tauri CLI version matches the Rust crate

3. **File Permissions (WSL Specific)**
   ```bash
   ls -la scripts/  # Ensure scripts are executable
   # Fix if needed: chmod +x scripts/*.sh
   ```

### Common WSL/Docker Gotchas

1. **Line Endings**: Always use LF, not CRLF
   ```bash
   # Check: file scripts/dev.sh
   # Fix: dos2unix scripts/*.sh
   ```

2. **Volume Mounting**: Ensure paths use WSL format
   - Good: `/mnt/c/Users/...`
   - Bad: `C:\Users\...`

3. **Performance**: Use Docker volumes for node_modules and target/
   - Already configured in docker-compose.yml
   - Prevents slow filesystem operations

## Code Generation Guidelines

### When Creating New Modules

1. **Start with Interface Design**
   ```rust
   // src/modules/new_feature/mod.rs
   pub trait FeatureInterface {
       // Define public API first
   }
   ```

2. **Implement Error Handling Early**
   ```rust
   // src/modules/new_feature/error.rs
   use thiserror::Error;
   
   #[derive(Error, Debug)]
   pub enum FeatureError {
       #[error("specific error: {0}")]
       SpecificError(String),
   }
   ```

3. **Add Tests Immediately**
   - Unit tests in the same file or tests.rs
   - Integration tests in tests/integration/

### Tauri-Specific Patterns

1. **Commands Must Be Async**
   ```rust
   #[tauri::command]
   async fn my_command(state: State<'_, AppState>) -> Result<String, String> {
       // Implementation
   }
   ```

2. **State Management**
   - Use Tauri's State for app-wide state
   - Implement Clone for state structs
   - Use Arc<Mutex<T>> for mutable shared state

3. **Frontend Communication**
   - Always use serde for serialization
   - Define TypeScript types matching Rust structs
   - Handle errors gracefully in both directions

### WASM Integration Points

1. **Binding Generation**
   ```rust
   // Always use wasm-bindgen proc macros
   use wasm_bindgen::prelude::*;
   
   #[wasm_bindgen]
   pub struct WasmModule {
       // Fields
   }
   ```

2. **Memory Management**
   - Minimize data copying between JS and WASM
   - Use typed arrays for large data transfers
   - Be aware of the JS garbage collector

## Testing Strategy

### Always Run These Tests

1. **Before Commits**
   ```bash
   cargo fmt --check
   cargo clippy -- -D warnings
   cargo test
   ```

2. **After Major Changes**
   ```bash
   cargo tauri build --debug  # Ensure it builds
   wasm-pack test --headless --chrome  # WASM tests
   ```

3. **Integration Points**
   - Test Tauri commands with frontend
   - Verify WASM module loading
   - Check cross-platform compatibility

## Performance Considerations

1. **Async Operations**
   - Use tokio for CPU-bound tasks
   - Keep UI thread responsive
   - Batch operations when possible

2. **Memory Usage**
   - Profile with `cargo flamegraph`
   - Watch for memory leaks in WASM
   - Use weak references where appropriate

3. **Build Times**
   - Use cargo workspaces effectively
   - Enable incremental compilation
   - Consider using sccache

## Security Checklist

- [ ] No hardcoded secrets or API keys
- [ ] Input validation on all Tauri commands
- [ ] CSP headers configured in tauri.conf.json
- [ ] Dependencies audited with `cargo audit`
- [ ] WASM modules sandboxed appropriately

## Common Commands Reference

```bash
# Development
./scripts/dev.sh start    # Start Docker environment
./scripts/dev.sh shell    # Enter container
cargo tauri dev          # Run in development mode

# Building
cargo build --release    # Optimized Rust build
wasm-pack build         # Build WASM modules
cargo tauri build       # Build desktop app

# Testing
cargo test              # Run all tests
cargo test -p module    # Test specific module
wasm-pack test         # Test WASM modules

# Code Quality
cargo fmt              # Format code
cargo clippy           # Lint code
cargo doc --open       # Generate docs
```

## When You Get Stuck

1. Check Docker logs: `docker-compose logs -f tauri-dev`
2. Verify all dependencies: `cargo tree`
3. Clear caches: `cargo clean && rm -rf target/`
4. Restart Docker: `./scripts/dev.sh stop && ./scripts/dev.sh start`
5. Check file permissions and line endings

## File Organization Patterns

When adding new functionality:
1. Create feature module in `src/modules/`
2. Define types in `types.rs`
3. Implement logic in `impl.rs` or split further
4. Export public API in `mod.rs`
5. Add integration tests in `tests/`
6. Update documentation in `docs/`

Remember: Modularity and the 800-line limit are not suggestions—they're requirements for maintainability.

---
> Source: [qLeviathan/testing_grounds](https://github.com/qLeviathan/testing_grounds) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
