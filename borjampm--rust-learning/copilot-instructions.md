## rust-learning

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This is a Rust learning repository. The user has a background in software engineering, low-level programming, and multiple languages. When helping:

- Always explain the **why** and **when** of Rust concepts, not just the how
- Compare/contrast with concepts from other languages when helpful
- Help create exercises with three parts: task description, main.rs file for coding, and tests to verify correctness

## Repository Structure

- `the-rust-book/projects/` - Practice projects following "The Rust Programming Language" book
- `the-rust-book/exercises/` - Exercises for practicing Rust concepts
- `the-rust-book/notes/` - Study notes
- `book/` - Reference copy of the Rust Book source (listings can be referenced but don't modify)

## Common Commands

```bash
# Build a project
cargo build                           # Debug build
cargo build --release                 # Release build

# Run a project
cargo run                             # Run default binary
cargo run --bin <name>                # Run specific binary

# Run tests
cargo test                            # Run all tests
cargo test <test_name>                # Run specific test
cargo test -- --nocapture             # Show println! output during tests

# Check code without building
cargo check                           # Fast compile check
cargo clippy                          # Linter with suggestions

# Format code
cargo fmt

# Open Rust documentation
rustup doc                            # Opens local Rust documentation
rustup doc --book                     # Opens "The Rust Programming Language" book
```

## Creating Exercises

When creating exercises:

1. Create a new cargo project in `the-rust-book/exercises/`:
   ```bash
   cargo new the-rust-book/exercises/<exercise-name>
   ```

2. Structure the exercise with:
   - A comment block at the top of `main.rs` describing the task
   - A skeleton `main.rs` with function signatures for the user to implement
   - Tests in the same file using `#[cfg(test)]` module
   - Avoid explaining too much to the user, make exercises a little open-ended to force learning.

Example exercise structure:
```rust
// Exercise: <brief description>
//
// <detailed task description>

fn solution_function() {
    todo!()
}

fn main() {
    // Optional: example usage
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_case_1() {
        // Test implementation
    }
}
```
3. After the user has finished, he will ask for comments on the solution. Provide at least:
    - Feedback on code quality
    - How rust is the code, and practices to improve it and be more idiomatic.
    - Things that were well.
    - Places where the code could be optimized.
    - Things that are not clear or confusing for someone that did not write the code.
    - Any other feedback that is relevant.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Borjampm)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Borjampm)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
