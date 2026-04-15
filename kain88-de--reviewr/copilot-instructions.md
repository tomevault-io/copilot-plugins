## reviewr

> This is a CLI tool to manage employee evaluations.

This is a CLI tool to manage employee evaluations.

## Development

To test the tool during development, you can use `cargo run`. For example:

```bash
cargo run -- add "John Doe"
cargo run -- notes "John Doe"
```

## Testing

To test the tool in a clean environment, you can use the `--data-path` flag to specify a directory for storing test data. This is useful for preventing your regular data from being affected by tests.

```bash
cargo run -- --data-path .test_eval_data/ add "John Doe"
cargo run -- --data-path .test_eval_data/ notes "John Doe"
```

## Quality Standards

This project enforces a high standard of code quality through a combination of automated checks and a strict build process.

### Formatting

All code must be formatted with `cargo fmt` before being committed. This ensures a consistent style throughout the project.

### Static Analysis

`cargo clippy` is used for static analysis to identify potential bugs and improve code quality. All Clippy warnings must be addressed before committing.

### Build Process

During the build process, all warnings are treated as errors. This is enforced by the following configuration in `.cargo/config.toml`:

```toml
[build]
rustflags = ["-D", "warnings"]
```

## Testing Philosophy

Our testing strategy is composed of two layers: high-level integration tests and low-level unit tests.

### Integration Tests

Our primary focus is on **behavior-driven integration testing**. We treat the application as a black box, interacting with it from the outside-in, just as a user would. This ensures that our tests verify the actual behavior of the application, rather than its internal implementation details.

**Core Principles:**

1.  **Test User Journeys:** Tests should mirror user actions. We will test the full execution path of a command, from the command line to the resulting output or side effects on the file system.
2.  **Avoid Mocks:** We will use real file systems and application binaries. Instead of mocking the file system, we will create temporary data directories for each test run, ensuring that tests are isolated and do not interfere with each other or the development environment.
3.  **High-Level Assertions:** Assertions will be made against the observable outputs of the application. This includes checking the exit code of the process, the content of `stdout` and `stderr`, and the state of files created or modified by the application.
4.  **Refactoring-Friendly:** By focusing on the external behavior of the application, our tests will remain valid even if the internal implementation is completely refactored. As long as the application's behavior remains the same, the tests will continue to pass.

### Unit Tests

While integration tests provide a high-level view of the application's behavior, unit tests allow us to test the functionality of individual modules in a more granular way. We will write unit tests for all public functions in our modules to ensure that they behave as expected.

## Pre-Commit Checks

Before committing any changes, the following checks must be performed:

1.  **Run all linters:** `just lint`
2.  **Run all tests:** `cargo test`

All checks must pass before a commit can be made. This ensures that the codebase remains in a clean and consistent state.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kain88-de) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
