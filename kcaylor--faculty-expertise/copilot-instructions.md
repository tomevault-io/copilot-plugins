## faculty-expertise

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands
- **Install dev mode**: `pip install -e .`
- **Export notebooks**: `nbdev_export`
- **Compile changes**: `nbdev_prepare`
- **Watch for changes**: `./watch.sh`

## Code Style Guidelines
- **Development flow**: Edit notebooks in `nbs/`, export to Python modules
- **Imports**: Standard imports at top, use relative imports for internal modules
- **Naming**: PascalCase for classes, snake_case for functions/variables
- **Documentation**: Documentation in notebooks, exported to modules
- **Error handling**: Use try/except with specific error messages
- **Data structures**: Use pandas DataFrames for tabular faculty data
- **Testing**: Inline tests in notebooks, no separate test framework

## Project Structure
- `nbs/`: Source notebooks (primary development location)
- `faculty_expertise/`: Exported Python modules (don't edit directly)
- `faculty_html/`, `faculty_screenshots/`: Raw data files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kcaylor) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
