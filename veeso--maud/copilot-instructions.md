## maud

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Maud is a Gleam library that renders Markdown into Lustre HTML components, inspired by MDX. It uses **mork** for Markdown parsing and **lustre** for the component/element model. Target platform is Erlang.

## Build Commands

```sh
gleam build              # Build the project
gleam test               # Run all tests
gleam format src test    # Format source and test files
gleam format --check src test  # Check formatting (used in CI)
gleam deps download      # Download dependencies
```

There is no single-test runner built in; test functions are any public function ending in `_test` in files under `test/`.

## Architecture

**Component-based rendering with user-overridable views:**

- `src/maud.gleam` — Public API. `render_markdown/3` parses a Markdown string via mork then delegates to `render_document/2` which walks the parsed document tree and produces `lustre/element.Element(a)` values.
- `src/maud/components.gleam` — Defines the `Components(a)` record type with 22 fields, each a function that produces a Lustre element for a specific Markdown construct (headings, paragraphs, links, code, etc.). `default()` returns standard HTML implementations. `with_*` functions allow overriding individual components.
- `src/internal/render.gleam` — Internal render logic (document tree → element tree conversion).

**Rendering model:** Bottom-up. Children are rendered first into `List(Element(a))`, then passed to the parent component function from `Components(a)`.

**Key dependencies:**
- `mork` — Markdown parser (produces `Document` AST)
- `lustre` — UI framework (provides `Element(a)` type and `html.*` functions)

## Gleam Conventions

- Use `gleam format` before committing (CI enforces `gleam format --check`)
- Test functions must be public and end with `_test`
- Testing framework: gleeunit

---
> Source: [veeso/maud](https://github.com/veeso/maud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
