## thm-restate

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is `thm-restate`, a Quarto extension that enables restating theorem environments in different parts of a document (e.g., state theorem in introduction, restate it in appendix with proof). Supports both HTML and PDF output.

## Build Commands

```bash
# Render example document to both HTML and PDF
quarto render example.qmd

# Render to specific format
quarto render example.qmd --to html
quarto render example.qmd --to pdf
```

## Architecture

### Two-Filter Design

Quarto's crossref system extracts `#thm-*` identifiers at the reader level before any Lua filters run. This requires a two-filter architecture:

1. **`thm-restate-early.lua`** - Runs BEFORE `quarto` filter
   - Captures theorem data via `restate-tag` attribute (since `#id` is already stripped)
   - Parses source file directly to extract theorem names from `## Heading` syntax
   - Stores captured data in document metadata (`meta["thm-restate-data"]`)

2. **`thm-restate.lua`** - Runs AFTER `quarto` filter
   - Reads theorem data from metadata
   - Processes `.restate` divs and renders format-specific output
   - LaTeX: Uses `\textbf{...}\itshape` for amsthm-like styling
   - HTML: Creates div with `.theorem .restated` classes

### Filter Ordering

Filters must be specified in the document YAML (not `_extension.yml`) to control ordering:

```yaml
filters:
  - _extensions/thm-restate/thm-restate-early.lua
  - quarto
  - _extensions/thm-restate/thm-restate.lua
```

### Usage Syntax

```markdown
::: {#thm-compact .restatable restate-tag="thm-compact"}
## Theorem Name
Content...
:::

::: {.restate ref="thm-compact"}
:::
```

The `restate-tag` attribute duplicates the ID because Quarto strips `#id` before filters run.

### Supported Theorem Types

Built-in Quarto types: `thm`, `lem`, `cor`, `prp`, `cnj`, `def`, `exm`, `exr`, `sol`, `rem`, `alg`

Custom types from `custom-amsthm` metadata are also supported.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brentonk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
