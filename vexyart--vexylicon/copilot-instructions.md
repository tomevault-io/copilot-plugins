## vexylicon

> Vexylicon is a Python package for creating sophisticated liquid-glass SVG icon effects. It transforms SVG icons with dual contours into glass-morphism designs with beveled edges and theme support.

# Vexylicon Development Guidelines

## Project Overview

Vexylicon is a Python package for creating sophisticated liquid-glass SVG icon effects. It transforms SVG icons with dual contours into glass-morphism designs with beveled edges and theme support.

**Current Status**: Alpha release - core glass effect working, theme system needs completion, web interface pending.

## Architecture Overview

```
vexylicon/
├── core.py              # VexyliconGenerator - main glass effect engine
├── cli.py               # Fire-based CLI interface
├── utils/
│   ├── svg_processor.py # lxml-based SVG DOM manipulation (NO string operations)
│   ├── path_tools.py    # Path interpolation algorithms from icon_blender.py
│   └── theme_loader.py  # Pydantic-based theme validation
└── assets/
    ├── best_base.svg    # Canonical base template with dual contours
    └── themes/
        └── default.json # Theme definitions
```

## Core Workflow

1. **Input**: `best_base.svg` (1200x1200 with dual contours) + optional payload SVG
2. **Processing**:
   - Parse dual contours (outer and inner paths)
   - Generate N interpolated rings (default 24)
   - Apply opacity progression (quartic by default)
   - Add `mix-blend-mode: screen` for glass effect
   - Inject payload with clipPath masking
3. **Output**: Glass effect SVG with optional theme variants

## Development Principles

### Code Quality
- Use `uv pip`, never `pip`
- Type hints on all public functions
- Docstrings in NumPy style
- Test coverage >90%
- Black formatting, ruff linting

### SVG Processing Rules
- **ALWAYS use lxml** for XML manipulation
- **NEVER use string replacement** for SVG operations
- Preserve namespaces and attributes
- Round coordinates to 2 decimal places

### Key Implementation Details

1. **Path Interpolation**: 
   - Reverse outer contour for proper alignment
   - Convert Line segments to degenerate CubicBeziers
   - Ensure segment count compatibility

2. **Opacity Progression**:
   - Mode 1: Linear
   - Mode 2: Decreasing (reverse exponential)
   - Mode 3: Exponential (quadratic)
   - Mode 4: More exponential (quartic) - DEFAULT

3. **Theme System** (needs completion):
   - Duplicate gradients with -light/-dark suffixes
   - Adjust opacity for dark variants (+20%)
   - Add CSS for prefers-color-scheme

## Current Issues

1. **Theme Implementation Incomplete**:
   - Gradients not properly duplicated
   - CSS for theme switching missing
   - Theme groups created but not functional

2. **Payload Masking**:
   - Works but needs refinement for complex SVGs
   - ViewBox handling could be improved

3. **Missing Features**:
   - No Gradio-lite web interface yet
   - Limited test coverage
   - No progress indicators for batch mode

## Development Workflow

1. **Before Changes**:
   - Read existing code thoroughly
   - Check PLAN.md and TODO.md
   - Understand the dual-contour SVG format

2. **Making Changes**:
   - Work in small, testable increments
   - Update CHANGELOG.md after each feature
   - Keep TODO.md current

3. **After Changes**:
   ```bash
   # Format and lint
   black src tests
   ruff check --fix src tests
   
   # Run tests
   pytest
   
   # Type check
   mypy src
   ```

## Priority Tasks

1. **Fix Theme System**:
   - Implement `_duplicate_gradients_for_themes()`
   - Add proper CSS with media queries
   - Test light/dark switching

2. **Gradio-lite Interface**:
   - Minimal dependencies (may need to vendor svgpathtools)
   - Handle Pyodide limitations
   - Focus on core features only

3. **Testing**:
   - Unit tests for all path_tools functions
   - Integration tests for full pipeline
   - Snapshot tests for SVG output

## Technical Constraints

- **Python 3.11+** required
- **Minimal dependencies**: lxml, svgpathtools, fire, pydantic, rich
- **No string manipulation** for SVG operations
- **Gradio-lite limitations**: Some packages won't work in browser

## File Path Tracking

Always include `# this_file: path/to/file.py` at the top of each Python file.

## Logging Guidelines

When adding logging (pending):
- Use loguru for structured logging
- Add `--verbose` flag to CLI
- Log at DEBUG level for development
- Include timing information for optimization

## Error Handling

- Custom exceptions: `VexyliconError`, `InvalidSVGError`, `ThemeValidationError`
- Provide actionable error messages
- Suggest fixes for common issues
- Validate inputs early

## Remember

- Focus on core functionality, not fancy features
- Glass effect quality is the top priority
- Keep dependencies minimal
- Test with real SVG files
- Document edge cases

$END$

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vexyart) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
