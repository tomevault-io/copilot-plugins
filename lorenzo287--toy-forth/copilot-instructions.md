## toy-forth

> This document provides foundational context, coding standards, and roadmap for Toy Forth. All AI agents MUST adhere to these guidelines to ensure consistency and system integrity.

# Toy Forth: Agent & Project Manual

This document provides foundational context, coding standards, and roadmap for Toy Forth. All AI agents MUST adhere to these guidelines to ensure consistency and system integrity.

## 1. Project Overview

Toy Forth: minimalist, stack-based interpreter in C. Dynamic object system (Integers, Floats, Strings, Lists, Symbols). Iterative execution engine: avoids C recursion for user words.

### Key Architectural Pillars

- **Iterative Engine**: Frame-based return stack (`tf_frame`) manages execution.
- **Execution Boundary**: User-defined words run through the explicit frame stack. Native words may still call `exec()` synchronously for quotation/control behavior; those blocking quotation runners should use the `_r` suffix until they are converted to continuation-style execution.
- **Refcounting**: Every object (`tf_obj`) managed via `retain_obj` and `release_obj`.
- **Global Dictionary**: High-performance hash table for word lookups.
- **Variable Capturing**: Dynamic scoping for locals via `{ a b }`.
- **Interactive REPL**: Persistent interpreter context across entries, with multiline input handling, colored diagnostics, and interrupt support.

## 2. Directory Structure

- `src/`: Core C implementation.
- `include/`: Internal API headers.
- `deps/`: Third-party sources bundled with the project (`linenoise`, `stb_leakcheck`).
- `fth/`: Forth scripts and tests.
- `tools/`: External tooling (Tree-sitter, LSP).
- `docs/`: Documentation for build, REPL usage, and external tools.

## 3. Coding Guidelines (C)

- **Style**: Snake_case, 4-space indentation.
- **Prefixing**: Use `tf_` prefix (e.g., `tf_obj`).
- **Memory Safety**:
  - `retain_obj` when storing reference.
  - `release_obj` when reference no longer needed.
  - Use `xmalloc`.
- **Verification**: Check for leaks with `stb_leakcheck`.

## 4. Agent Interaction Rules

- **Proactive Testing**: Propose/implement tests in `fth/` for fixes/features.
- **Surgical Edits**: Minimal, precise changes only.
- **Context Awareness**: Read headers in `include/` before engine/lexer edits.
- **Native Naming**: Use `_r` for native words that call `exec()` and wait for a quotation result. Do not use `_r` for natives that only manipulate the data stack, register definitions, or schedule a frame and return.
- **C-First Workflow**: For implementation sessions, focus primarily on the C interpreter and Forth behavior. If a new native word or syntax change affects tooling, keep metadata updates lightweight and obvious, but do not spend time on full Tree-sitter/LSP/VS Code verification unless the task is specifically about tooling or the user requests it.
- **Language Direction**: Prefer the quotation-first model described in `docs/language-roadmap.md`. Treat user-defined words as named quotations, keep `: ... ;` as compatibility sugar unless a task explicitly changes that policy, and prioritize list/quotation primitives and combinators over new syntax.
- **Concise Communication**: Be concise by default. Expand when needed for clarity, reasoning, or uncertainty. Avoid filler, not explanation.
- **Execution Context**: Assume Windows PowerShell. Do not output bash syntax.

## 5. Tooling Roadmap

1. **Tree-sitter Grammar**: [Done] Formal grammar in `tools/tree-sitter-toyforth/`.
2. **Language Server Protocol (LSP)**: [Done] Standalone Go implementation in `tools/toyforth-lsp/`.
3. **VS Code Extension**: [Done] Syntax highlighting and LSP client in `tools/vscode-toyforth/`.
4. **REPL Enhancements**: [Done] Interactive REPL with multiline input, colored diagnostics, `Ctrl-C` interruption handling, and `linenoise` history/completion. See `docs/repl.md`.
5. **Formatter**: Auto layout based on Tree-sitter.
6. **Integrated Debugger**: Step-by-step execution and stack inspection.

## 6. Build & Verification

### Environment

- OS: Windows
- Shell: PowerShell (commands assume PowerShell syntax, not bash)
- Compiler: (e.g., MSVC via CMake or specify if using MinGW/Clang)
- Paths: Use Windows-style paths unless otherwise specified

### Build Commands

Predefined build directories:

- **Standard**: `cmake --build build`
- **Leak Check**: `cmake --build build-leak` (uses `stb_leakcheck`)
- **Profiling**: `cmake --build build-prof`

Default coding-session verification:

- Build the C target with `cmake --build build`.
- Run the relevant `fth/` script(s), plus a broader script when risk justifies it.
- Use `build-leak` for changes that alter ownership, object lifetime, stack effects, or execution flow.
- Tooling tests (`go test`, Tree-sitter generation/tests, VS Code checks) are optional by default; run them only for tooling-focused tasks, risky tooling edits, or explicit user requests.

### Documentation

- REPL usage and behavior: `docs/repl.md`
- Build instructions: `docs/build.md`
- Language roadmap: `docs/language-roadmap.md`
- Tree-sitter: `docs/tree-sitter.md`
- LSP: `docs/lsp.md`
- VS Code extension: `docs/vscode.md`

---
> Source: [Lorenzo287/toy_forth](https://github.com/Lorenzo287/toy_forth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
