## tex2html

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

tex2html is a Python wrapper around LaTeXML that converts LaTeX documents to HTML (and other formats via pandoc). Features a filesystem-based **theme + component system** where themes provide CSS variables and components add interactive functionality (CSS + optional JS). Zero Python runtime dependencies — uses only stdlib and delegates to external tools (LaTeXML, pandoc).

The primary consumer is `mf papers process`, which calls `tex2html {file} -f html5 -o {output_dir}`.

## Development Commands

```bash
# Install with dev dependencies
pip install -e ".[dev]"

# Run all tests (coverage enabled by default via pyproject.toml)
pytest

# Run a single test file
pytest tests/test_components.py

# Run a specific test
pytest tests/test_components.py::TestLoad::test_loads_css -v

# Format code (Black, line length 88)
black src/

# Type checking
mypy src/

# Linting
flake8 src/
```

### CLI Testing

```bash
# Basic conversion (requires LaTeXML installed)
tex2html document.tex

# With theme + components
tex2html document.tex --theme academic -c dark-mode,floating-toc

# List available options
tex2html --list-themes
tex2html --list-components
```

## Architecture

### Conversion Pipeline

```
HTML formats:  .tex → latexmlc → .html → compose() (theme + components) → final .html
Other formats: .tex → latexmlc → .html → pandoc → .md/.txt/.epub
XML format:    .tex → latexml → .xml (no post-processing)
```

### Core Modules

- **`converter.py`** — `TexConverter` class wraps LaTeXML, routes to format-specific converters via `_FORMAT_DISPATCH` table. Entry point: `convert(format, **kwargs)`. After HTML conversion, calls `compose()` to inject theme/components.
- **`composer.py`** — Free functions (`compose()`, `inject_into_head()`, `inject_before_body_close()`) that modify HTML in-place on disk. Uses regex to find `</head>` and `</body>` insertion points. Special handling: `dark-mode` component gets an early inline script to prevent flash of unstyled content.
- **`themes.py`** — Auto-discovers themes from `data/themes/*.css`. Each `.css` file becomes a `Theme` entry, with description parsed from the first CSS comment. No manual registration needed.
- **`components.py`** — Auto-discovers components from `data/components/*.css`. A component = required `.css` + optional `.js`. Supports an `extra_dir` that shadows built-in components. Path traversal is rejected.
- **`cli.py`** — argparse CLI. Validates component names against available set before starting conversion.

### Auto-Discovery Pattern

Both themes and components use filesystem-based auto-discovery — no registry to maintain. Adding a new theme or component is just adding files:

**New theme:** Create `src/tex2html/data/themes/mytheme.css` with a `/* Description */` first-line comment. It appears automatically in `--list-themes` and `--theme` choices.

**New component:** Create `src/tex2html/data/components/mycomp.css` (and optionally `mycomp.js`). It appears automatically in `--list-components` and can be used with `-c mycomp`.

### Theme-Component Integration

Themes define CSS variables; components consume them with fallback defaults:
```css
/* Theme defines */   :root { --toc-bg: #f8f8f8; }
/* Component uses */  .floating-toc { background: var(--toc-bg, #f8f8f8); }
```

## System Dependencies

- **LaTeXML** (required): `sudo apt-get install latexml` or `brew install latexml`
- **Pandoc** (optional, for markdown/txt/epub): `sudo apt-get install pandoc`

## Key Constraints

- **Python ≥ 3.10** — uses `X | None` union syntax
- **Zero runtime dependencies** — stdlib only
- `composer.py` uses regex for HTML manipulation (fragile, should use `html.parser`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/queelius) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
