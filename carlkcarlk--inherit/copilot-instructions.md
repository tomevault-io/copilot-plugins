## inherit

> This repository is a Rust tutorial project focused on inheritance-like patterns.

# Coding Notes for Agents

This repository is a Rust tutorial project focused on inheritance-like patterns.

## Scope

- Keep examples small, readable, and teaching-first.
- Prefer clarity over cleverness.
- Keep puzzle markdown (`*_puzzle.md`) concept-first and language-agnostic when possible.
- Keep solution `.rs` files explicit about Rust mechanics.

## Naming

- Use descriptive names.
- Avoid abbreviations in variable names.
  - use `addresses`, not `addrs`.
  - use `credentials`, not `creds`.
- Avoid single-letter variable names except very short local loop counters when unavoidable.
- Match variable names to type names in snake_case when practical.

## Rust Style

- Prefer `use` imports over long fully-qualified paths in code.
- Prefer `new()` constructors for local example types.
- Avoid unnecessary `unsafe`. If used, explain why.
- Avoid silent clamping; prefer explicit checks/assertions.

## Examples

- Examples should compile with `cargo check --example <name>`.
- Avoid warnings in examples when practical.
- Include short comments only when they clarify a teaching point.
- Show one concept per example file.

## Puzzle Markdown

- Mermaid diagrams should emphasize OO mental models.
- Use `is-a` labels for inheritance-style arrows in puzzle diagrams.
- Keep puzzle text concise.
- Keep Rust-specific caveats in solution files, not puzzle files.

## Editing Rules

- Do not create `mod.rs` files.
- Do not remove user-authored comments unless they are incorrect; prefer editing for accuracy.
- Preserve existing style and file naming patterns in `examples/`.

## Verification

- For single example changes: run `cargo check --example <name>`.
- For broader validation: run `cargo check --examples --all-targets`.

---
> Source: [CarlKCarlK/inherit](https://github.com/CarlKCarlK/inherit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
