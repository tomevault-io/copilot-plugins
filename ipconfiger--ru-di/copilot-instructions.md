## ru-di

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ru-di is a lightweight dependency injection (DI) container library for Rust, providing both synchronous and asynchronous DI implementations. The entire implementation is contained in a single file (`src/lib.rs`).

**Published crate**: [ru-di on crates.io](https://crates.io/crates/ru-di)

## Development Commands

```bash
# Build the project
cargo build

# Run all tests (both sync and async)
cargo test

# Run only sync tests
cargo test it_works

# Run only async tests
cargo test async_test

# Check code without building
cargo check

# Format code
cargo fmt

# Run linter
cargo clippy
```

## Architecture

### Dual Container Design

The library provides two separate containers for different programming paradigms:

1. **`Di` (Synchronous)**: Uses `std::sync` primitives (`Mutex`, `RwLock`)
   - Factory registration: `Fn(&Di) -> T` (has access to container for nested dependencies)
   - Methods: `register()`, `register_single()`, `get()`, `get_single()`

2. **`TkDi` (Asynchronous)**: Uses `tokio::sync` primitives (`TokioMutex`, `TokioRwLock`)
   - Factory registration: `Fn() -> T` (simplified, no container access)
   - All methods are `async`
   - Methods: `register()`, `register_single()`, `get()`, `get_single()`

### Singleton Pattern

Both containers support thread-safe singleton services:
- **Sync**: `SingleRef<T>` wrapping `Arc<RwLock<T>>`
- **Async**: `SingleAsyncRef<T>` wrapping `Arc<TokioRwLock<T>>`
- Both provide `get()` (read lock) and `get_mut()` (write lock) methods

### Type Erasure

Uses `TypeId` and `dyn Any` for runtime type-safe storage:
- Providers stored in `HashMap<TypeId, Arc<dyn Provider>>`
- Singletons stored with type-safe retrieval via unsafe pointer casting (see lib.rs:224-228)

### Global State

- Global singleton containers initialized via `OnceLock`
- `STD_INSTANCE`: Static `Mutex<Di>` for sync operations
- `AYN_INSTANCE`: Static `TokioMutex<TkDi>` for async operations

## Key Implementation Details

### Thread Safety

- All operations are thread-safe using appropriate synchronization primitives
- Lock operations can fail (returns `DiError::LockError`)
- Use `.unwrap()` carefully in tests, but handle errors properly in production code

### Unsafe Type Conversion

The library uses unsafe pointer casting to convert between erased types (lib.rs:224-228, 136-140):
```rust
let value = unsafe {
    let ptr = Arc::into_raw(any.clone());
    Arc::from_raw(ptr as *const RwLock<T>)
};
```
This is safe because the type ID is verified before conversion.

### Provider Traits

- **`Provider`**: Sync trait with `provide(&self, di: &Di) -> Box<dyn Any>`
- **`TkProvider`**: Async trait with `provide(&self) -> Box<dyn Any>`
- **`FactoryProvider`**: Generic struct implementing both traits for different factory signatures

## Testing Strategy

- All tests are inline in `src/lib.rs` using `#[cfg(test)]`
- Tests serve as usage examples
- Both sync (`it_works`) and async (`async_test`) test functions
- Use tokio runtime for async tests: `#[tokio::test]`

## API Design Patterns

### Registration
- Transient services: Register with factory closure
- Singleton services: Register with instance
- Sync factories receive `&Di` parameter for resolving dependencies
- Async factories don't receive container (simplified API)

### Retrieval
- Transient: Returns `Result<T, Box<dyn Error>>`
- Singleton: Returns `Option<SingleRef<T>>` or `Option<SingleAsyncRef<T>>`

## Code Conventions

- Keep implementation simple and focused on core DI functionality
- No premature abstractions - single file is acceptable for this scope
- Thread safety is mandatory for all operations
- Handle all error cases explicitly (no silent failures)
- Tests should demonstrate real usage patterns

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ipconfiger) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
