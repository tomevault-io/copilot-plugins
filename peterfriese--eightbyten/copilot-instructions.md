## eightbyten

> **EightByTen** is a premium Tufte-style book template for Typst. It focuses on aesthetics, providing a "batteries-included" experience for authors who want beautiful, wide-margin layouts with semantic structure.

# EightByTen Package Agent Guide

## Overview

**EightByTen** is a premium Tufte-style book template for Typst. It focuses on aesthetics, providing a "batteries-included" experience for authors who want beautiful, wide-margin layouts with semantic structure.

## Core Philosophy

1.  **Premium Aesthetics**: The output should wow the user. Typography, spacing, and layout are paramount.
2.  **Semantic Structure**: We provide high-level abstractions (`#part`, `#chapter`, `#note`) rather than low-level formatting tools.
3.  **Battery-Included**: The package should work out of the box with minimal configuration, but remain flexible via `typst.toml` and `config.typ`.

## Directory Structure

*   `src/`: **The Source of Truth**. Contains the actual package code.
    *   `lib.typ`: The entry point for the package. Exposes the API.
    *   `template.typ`: The main show rule (`eightbyten`).
    *   `layout.typ`: Page setup, margins, and geometry logic.
    *   `headers.typ`: Header and footer generation logic.
    *   `structs.typ`: Definitions for `#part`, `#chapter`, `#note`, etc.
    *   `config.typ`: Default configuration values (colors, fonts).
*   `template/`: A reference implementation using the package. This is what the user copies to start their book.
*   `scripts/`: Bash scripts for packaging and installation.
*   `thumbnails/`: Generated images for the repository README.

## Development Workflow

We use `just` (Justfile) to manage development tasks.

### 1. Installation

To install the current version of the package locally for development:

```bash
just install
```

This installs the package to `@local/eightbyten:<version>`. The version is read from `typst.toml`.

To install to the preview namespace (useful for pre-release testing):

```bash
just install-preview
```

### 2. Verification

**ALWAYS** verify changes by compiling the template. The template is our integration test.

```bash
just verify-template
```

This command:
1.  Installs the current package to `@local`.
2.  Compiles `template/main.typ` using the `@local` package.

If this fails, **do not commit**.

### 3. Thumbnails

If you make layout changes that affect the visual appearance, regenerate the thumbnails:

```bash
just thumbnails
```

### 4. Code Style

*   **Imports**: Use standard Typst imports. Inside `template/`, use `@local` imports to reference the package.
*   **Commits**: Use **Conventional Commits** (e.g., `feat:`, `fix:`, `docs:`, `chore:`). Keep commits atomic.

## Common Tasks

### Adding a New Feature
1.  Implement the logic in a new or existing file in `src/`.
2.  Export it in `lib.typ`.
3.  Update `template/main.typ` or other template files to demonstrate the feature.
4.  Run `just verify-template`.
5.  Document it in `README.md` and `template/AGENTS.md`.

### Updating Dependencies
1.  Modify `typst.toml`.
2.  If adding a new tool (like `codly`), ensuring it is properly initialized in `template.typ`.

## Troubleshooting

*   **"Package not found"**: Run `just install` to ensure the registry is up to date.
*   **"Layout did not converge"**: This is common with complex Tufte layouts. Check `layout.typ` for circular dependency in state queries (e.g., headers measuring themselves).
*   **"Script hang"**: The `scripts/package` script uses `rm -rf` to clean up. If it hangs, check file permissions in `~/Library/Application Support/typst/packages/local`.

## States maintained

- `chapter-eyebrow` — the chapter's eyebrow text. Set by `#chapter()`, reset by `#part()`, `#appendix()`, `#special-appendix()`, `#appendix-part()`. Read by `chapter-page-layout` in `layout.typ`.
- `chapter-numbering` — the current chapter's numbering style (`auto`, `none`, or a custom numbering function). Set by `#chapter()`, `#appendix()`, `#special-appendix()`, `#part()`. Read by `#section()`, `#subsection()`, and `#subsubsection()` to decide whether to suppress section numbering in unnumbered chapters.

---
> Source: [peterfriese/eightbyten](https://github.com/peterfriese/eightbyten) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
