## rustanimations

> - `cargo check` - Quick syntax/type checking

# AGENTS.md - Rust Bevy Animation Project

## Build/Test Commands
- `cargo check` - Quick syntax/type checking
- `cargo build` - Build project
- `cargo run` - Run the animation application
- `cargo test` - Run all tests
- `cargo test <test_name>` - Run specific test by name
- `cargo clippy` - Lint code
- `cargo fmt` - Format code

## Project Structure
- `src/lib.rs` - Library root with public API
- `src/main.rs` - Application entry point
- `src/components.rs` - Bevy components
- `src/config.rs` - Configuration resources
- `src/systems/` - Bevy systems (animation, rendering, setup)
- `src/movements/` - Physics and particle systems
- `src/shapes/` - Geometric shapes and utilities
- `src/testing/` - Test modules with #[cfg(test)]
- `src/error.rs` - Error types and handling

## Code Style & Conventions
- **Module structure**: Use `mod.rs` files, expose public items via `pub use`
- **Imports**: Group std library first, then external crates, then local modules
- **Naming**: snake_case for functions/variables, PascalCase for types/structs
- **Types**: Use explicit types for struct fields (f32, not float)
- **Error handling**: Use custom `AnimationError` type and `Result<T, AnimationError>`
- **Testing**: Use proper `#[test]` attributes, tests in `#[cfg(test)]` modules
- **Bevy patterns**: Separate systems for different concerns, use Resources for configuration
- **Documentation**: Use `///` for public API docs, document components and resources
- **Formatting**: Use `cargo fmt` - 4-space indentation, max 100 char lines

---
> Source: [JoJoDataPhysics/rustanimations](https://github.com/JoJoDataPhysics/rustanimations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
