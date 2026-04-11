## prescient

> This project is a compile-time optimized Entity Component System (ECS) for the Zig programming language. The architecture is designed for high performance by leveraging Zig's compile-time features to generate specialized code for different component combinations (archetypes).

# GEMINI.md

## Project Overview

This project is a compile-time optimized Entity Component System (ECS) for the Zig programming language. The architecture is designed for high performance by leveraging Zig's compile-time features to generate specialized code for different component combinations (archetypes).

The core of the system is the `ArchetypePool`, which is a compile-time defined collection of archetypes that share a common set of possible components. This allows for semantic grouping of entities and optimization of storage and access patterns.

The ECS prioritizes:

*   **Compile-time safety:** Most type checking and validation is done at compile-time.
*   **Performance:** The system is designed to be cache-friendly and avoid runtime overhead.
*   **Flexibility:** While most things are defined at compile-time, archetypes are created at runtime as needed.

## Building and Running

The project uses the Zig build system. The following commands are available:

*   **Build:** `zig build`
*   **Run:** `zig build run`
*   **Test:** `zig build test`

## Development Conventions

The codebase is well-structured and follows Zig's idiomatic style. The `README.md` file is very detailed and serves as the primary source of documentation for the project's architecture and design philosophy.

Key conventions include:

*   **Compile-time configuration:** Components, pools, and queries are defined at compile-time.
*   **Immutability:** The ECS is designed to be mostly immutable, with changes being handled through a migration queue.
*   **Explicitness:** The system is designed to be explicit about its operations, with clear boundaries between different parts of the system.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/austinrtn)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/austinrtn)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
