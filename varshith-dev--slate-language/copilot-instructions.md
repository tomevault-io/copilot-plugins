## slate-language

> This document outlines the architectural context, coding standards, and directives for any AI assistant or agent working within the Slate codebase. Slate is a universal visual scripting language that compiles to a JSON AST, which is then natively rendered in the browser.

# Claude and AI Assistant Instructions for Slate

This document outlines the architectural context, coding standards, and directives for any AI assistant or agent working within the Slate codebase. Slate is a universal visual scripting language that compiles to a JSON AST, which is then natively rendered in the browser.

## Architectural Imperatives

1. No HTML in Compiler: The Rust compiler (`src/compiler.rs`) must exclusively output the structured JSON AST. Do not introduce HTML template strings, inline CSS, or SVG generation within the Rust backend. 
2. Native DOM Construction: The `slate.js` engine handles all rendering. It must construct DOM elements programmatically using `document.createElement`. Do not use `.innerHTML` with concatenated HTML strings for security and structural integrity reasons.
3. Separation of Concerns: The Rust compiler parses syntax and manages the grammar rules. The JavaScript engine interprets the JSON AST and applies styling and layout logic (like flowchart topological sorting).
4. No Regular Expressions: The Slate lexer (`src/lexer.rs`) and parser (`src/parser.rs`) are implemented as recursive descent systems operating on a character level. Do not introduce Regex implementations for parsing syntax.

## Design System Directives

The `slate.js` engine must adhere strictly to a premium, flat, monochrome grayscale design system.
1. Strictly Grayscale: Never introduce vibrant colors. Use the predefined constants in `slate.js` (`C.text`, `C.muted`, `C.subtle`, `C.border`, `C.bg`).
2. Flat Design: Never introduce drop shadows (`box-shadow`), gradients, or complex bevels. Elements rely on borders and padding for definition.
3. Font Stacks: Use standard system UI fonts for text elements and monospace fonts for technical or data-driven labels, as defined in `slate.js`.

## Code Modification Rules

- When modifying the grammar in `parser.rs`, you must also update the AST definition in `ast.rs` and the JSON serialization logic in `compiler.rs`.
- When adding a new tag or node type, you must simultaneously add the corresponding rendering switch case in the `renderNode` function of `slate.js`.
- Always verify that new properties added to `Config` structures in the AST are properly escaped when serialized to JSON to prevent parser breaks.

## Testing and Verification

- To verify backend changes, compile a `.slt` script using `cargo run --release -- compile <file.slt>` and inspect the output JSON.
- Ensure the JSON is structurally sound and parses cleanly in JavaScript without syntax errors.
- Always execute `cargo build --release` after making Rust changes. Do not leave the repository in a broken compilation state.

---
> Source: [varshith-dev/slate-language](https://github.com/varshith-dev/slate-language) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
