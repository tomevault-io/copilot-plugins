## text-extractor

> Auto-generated from all feature plans. Last updated: 2025-09-30

# kris-extractor Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-09-30

## Active Technologies
- Python 3.10+ + kinter (built-in), tkinterdnd2 0.3.0+, PyMuPDF 1.23.0+ (PDF parsing), python-docx 1.1.0+ (DOCX parsing), PyInstaller 5.13+ (executable packaging) (001-use-docs-prd)
- JSON configuration file (config.json), keyword history persistence (002-i-want-to)
- Python 3.10+ (pyproject.toml targets py310) + kinter (GUI framework), tkinterdnd2 (drag-and-drop), PyMuPDF/python-docx (document parsing) (005-i-want-the)
- JSON file for keyword history persistence (KeywordHistory model) (005-i-want-the)
- Python 3.10+ + kinter (built-in), tkinterdnd2 0.3.0+, PyMuPDF 1.23.0+, python-docx 1.1.0+, olefile 0.46+ (007-i-want-the)
- JSON files (config.json for keyword history persistence) (007-i-want-the)
- Python 3.10+ (existing pyproject.toml target) + kinter (built-in), existing Configuration/ConfigurationManager infrastructure (008-docs-presets-prd)
- JSON file (config.json) - extends existing configuration persistence (008-docs-presets-prd)

## Project Structure
```
src/
tests/
```

## Commands
cd src [ONLY COMMANDS FOR ACTIVE TECHNOLOGIES][ONLY COMMANDS FOR ACTIVE TECHNOLOGIES] pytest [ONLY COMMANDS FOR ACTIVE TECHNOLOGIES][ONLY COMMANDS FOR ACTIVE TECHNOLOGIES] ruff check .

## Code Style
Python 3.10+: Follow standard conventions

## Recent Changes
- 008-docs-presets-prd: Added Python 3.10+ (existing pyproject.toml target) + kinter (built-in), existing Configuration/ConfigurationManager infrastructure
- 007-i-want-the: Added Python 3.10+ + kinter (built-in), tkinterdnd2 0.3.0+, PyMuPDF 1.23.0+, python-docx 1.1.0+, olefile 0.46+
- 005-i-want-the: Added Python 3.10+ (pyproject.toml targets py310) + kinter (GUI framework), tkinterdnd2 (drag-and-drop), PyMuPDF/python-docx (document parsing)

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Source: [smeshko/text-extractor](https://github.com/smeshko/text-extractor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
