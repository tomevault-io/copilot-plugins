## mcp

> Rules for Rust projects and toolchain configuration

# Rust Project Rules

## Requirements
- Use Rust version 1.85.1
- Follow Rust style guide
- Use Clippy linting
- Enable all warnings
- Treat warnings as errors
- we use the rustfmt formatter
- USE COMMENTS, but brief and to the point
- use nix build
- use nix run
- use nix develop
- YOUR MUST FIX LINTER ERRORS BEFORE YOU COMMIT
- IGNORING WARNINGS IS PERMITTED PRIOR TO INITIAL RELEASE

## Project Structure
- src/main.rs for binary
- src/lib.rs for library
- tests/ for integration tests
- benches/ for benchmarks
- examples/ for examples

## Toolchain Components
- rust-toolchain.toml required
- Cargo.toml required
- Cargo.lock committed
- Use the oxalica overlay for packages
- Use buildRustPackage for dependencies
- DO NOT USE naersk
- .gitignore for Rust

## Code Quality
- Document public items
- Write unit tests
- Use type safety
- Handle errors properly
- Follow SOLID principles
- use functional programming principles
- use async programming principles
- use reactive programming principles
- we use subject-based messaging patterns
- we use event-driven architecture

## Dependencies
- Pin dependency versions
- Audit dependencies
- Minimize dependencies
- Use workspace if needed

## Testing
- Write unit tests
- Add acceptance tests in gherkin format
- Include doc tests
- Use test fixtures
- Mock external services

## Error Handling
- Use Result type
- Handle all errors
- Provide context
- Create custom errors
- Log appropriately

## Performance
- Avoid allocations
- Use iterators
- Profile code
- Benchmark critical paths
- Cache when needed

## Security
- Use safe Rust
- Audit unsafe code
- Validate input
- Handle secrets properly
- Update dependencies

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TheCowboyAI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
