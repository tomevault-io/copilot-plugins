## my-cv

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is an academic CV generation system that creates LaTeX documents from YAML data files. The system uses Jinja2 templating to generate multiple CV formats (full CV, biosketch, grants documents) from structured YAML data and BibTeX bibliography files.

## Architecture

### Core Components

- `generate_cv.py`: Main CV generation script that processes YAML data and renders LaTeX templates
- `filters.py`: Jinja2 filters for LaTeX escaping and data manipulation
- `_config.yml`: Central configuration file defining personal info, file paths, and document sections
- `templates/`: LaTeX templates with custom Jinja2 delimiters (`((*`, `*))`, `(((`, `)))`)
- `my-cv-data/` (submodule): Private repository containing all CV data in YAML format
- `my-papers/` (submodule): Public repository containing BibTeX bibliography

### Data Structure

CV data is organized into YAML files in the `my-cv-data/` submodule:

- `education.yml`, `appointments.yml`, `grants.yml`, etc.
- Each file corresponds to a CV section defined in `_config.yml`
- Data fields are documented in `my-cv-data/README.md`

### Template System

- Uses custom Jinja2 delimiters to avoid LaTeX conflicts
- Templates generate to `temp_output/` directory
- Selected outputs copied to `docs/` for GitHub Pages

## Common Commands

### Build CV Documents

```bash
# Generate all CV templates (TeX files)
uv run python generate_cv.py

# Generate grants Excel file
uv run python grants_to_excel.py

# Run tests
uv run pytest
```

### Environment Setup

```bash
# Install dependencies (requires Python >=3.12, managed by uv via pyproject.toml)
uv sync
```

### PDF Compilation

The GitHub Actions workflow handles PDF compilation using XeLaTeX, but for local testing:

```bash
# In temp_output/ directory
xelatex -bibtex-cond1 Doss-Gollin-CV.tex
```

### Submodule Management

```bash
# Update bibliography submodule
git submodule update --remote my-papers

# Update CV data submodule (private)
git submodule update --remote my-cv-data
```

## Development Notes

### Template Configuration

- CV templates defined in `TEMPLATES` dict in `generate_cv.py`
- `copy_to_docs` flag controls which outputs go to GitHub Pages
- Full CV is public, biosketch and grants remain private

### Data Validation

- YAML data uses consistent date formats (YYYY for years, YYYY-MM-DD for dates)
- Monetary amounts use underscore separators (e.g., `75_000`)
- Grant data includes status tracking and collaborative project flags

### GitHub Actions

- Automatically builds PDFs on push to main branch
- Updates both submodules before building (uses token authentication for private submodule)
- Commits generated files back to repository
- Uses XeLaTeX for PDF compilation with bibtex processing
- Uses `uv` for Python dependency management (via `astral-sh/setup-uv` action)

### File Organization

- `temp_output/`: Generated TeX and PDF files
- `docs/`: Public files for GitHub Pages (CV only)
- LaTeX templates use `.latextemplate` extension

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jdossgollin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
