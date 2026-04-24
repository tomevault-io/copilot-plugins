## dbtbrew

> You are an expert in Rust, async programming, and concurrent systems.

# Rust Best Practices

You are an expert in Rust, async programming, and concurrent systems.

## Code Organization

- Use `lib.rs` for library crates and `main.rs` for binary crates.
- Organize code into modules with clear responsibilities - try to follow domain driver design (DDD) or clean architecture (by uncle Bob) patterns
- Keep functions short and focused on a single task.

## Error Handling

- Use `Result` and `Option` types for error and absence handling.
- Implement the `?` operator for propagating errors.
- Provide meaningful error messages.

## Naming Conventions

- Use `snake_case` for functions and variables.
- Use `CamelCase` for types and structs.
- Prefix boolean variables with `is_`, `has_`, `can_` or `should_`.

## Performance

- Prefer iterators over loops for data processing.
- Avoid unnecessary cloning; use references where possible.
- Leverage Rust's ownership model to manage resources efficiently.

## Async Programming

- Use `tokio` as the async runtime for handling asynchronous tasks and I/O.
- Implement async functions using `async fn` syntax.
- Leverage `tokio::spawn` for task spawning and concurrency.
- Use `tokio::select!` for managing multiple async tasks and cancellations.
- Favor structured concurrency: prefer scoped tasks and clean cancellation paths.
- Implement timeouts, retries, and backoff strategies for robust async operations.

## Channels and Concurrency

- Use Rust's `tokio::sync::mpsc` for asynchronous, multi-producer, single-consumer channels.
- Use `tokio::sync::broadcast` for broadcasting messages to multiple consumers.
- Implement `tokio::sync::oneshot` for one-time communication between tasks.
- Prefer bounded channels for backpressure; handle capacity limits gracefully.
- Use `tokio::sync::Mutex` and `tokio::sync::RwLock` for shared state across tasks, avoiding deadlocks

## Error Handling and Safety

- Embrace Rust's Result and Option types for error handling.
- Use `?` operator to propagate errors in async functions.
- Implement custom error types using `thiserror` or `anyhow` for more descriptive errors.
- Handle errors and edge cases early, returning errors where appropriate.
- Use `.await` responsibly, ensuring safe points for context switching.

## Testing

- Write unit tests for individual functions.
- Use integration tests in the `tests/` directory for end-to-end scenarios.
- Employ `#[cfg(test)]` to include test code conditionally.
- Write unit tests with `tokio::test` for async tests.
- Use `tokio::time::pause` for testing time-dependent code without real delays.
- Implement integration tests to validate async behavior and concurrency.
- Use mocks and fakes for external dependencies in tests.

## Documentation

- Document public APIs with `///` comments.
- Provide examples for complex functions or modules.

## Clean Code Principles

- Adhere to the Single Responsibility Principle: each module/function should have one responsibility.
- Avoid code duplication; extract common logic into reusable functions.
- Keep the codebase DRY (Don't Repeat Yourself).
- Ensure code readability and maintainability.
  

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nllahat) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
