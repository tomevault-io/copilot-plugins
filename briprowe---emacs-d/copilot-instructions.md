## emacs-d

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a personal Emacs configuration repository with a modular architecture. The configuration is designed for software development with support for multiple languages and modern development tools.

## Architecture

### Core Files
- `init.el`: Main entry point that sets up package management and loads configurations
- `custom.el`: Emacs custom variables and package selections (auto-generated, version-controlled)
- `ui.el`: UI-specific configurations (disables GUI elements, configures display)
- `mac.el`: macOS-specific settings

### Package Organization
- Individual package configurations are stored in `packages/*.el`
- Each file uses `use-package` declarations for consistent configuration
- Packages are automatically loaded from the `packages/` directory
- Mix of MELPA-stable packages and Git-based packages (via `:vc`)

## Key Development Patterns

1. **Adding New Packages**: Create a new file in `packages/` using the `use-package` pattern
2. **Package Sources**: Can use either MELPA or direct Git repositories (many custom forks are used)
3. **Performance**: Configuration is optimized for fast startup with lazy loading

## Language-Specific Configurations

### Rust
- Uses eglot for LSP support
- Auto-formatting on save enabled
- Treesitter integration for syntax highlighting

### Python
- Configured to use `uv` as the Python interpreter
- Flycheck for linting

### Other Languages
- Clojure/CIDER for Lisp development
- Web development tools (web-mode, CSS, JavaScript)
- SQL formatting via pgformatter
- Support for Julia, Zig, and others

## Important Notes

- This is a personal configuration without formal build/test infrastructure
- No Makefile or automated testing
- Heavy use of Git-based package installation from specific repositories
- Many packages are custom forks (e.g., `briprowe/cargo.el`)
- Uses Copilot integration for AI assistance

## Common Tasks

To reload configuration after changes:
- Restart Emacs, or
- Evaluate specific expressions with `M-x eval-buffer` in the changed file

To add a new package:
1. Create a new file in `packages/` directory
2. Use the `use-package` pattern (see existing files for examples)
3. Add package to `package-selected-packages` in `custom.el` if from MELPA
4. For Git-based packages, add to `package-vc-selected-packages`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/briprowe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
