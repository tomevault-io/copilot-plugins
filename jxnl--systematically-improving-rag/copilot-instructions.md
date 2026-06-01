## project-structure

> This is a documentation and workshop project for RAG (Retrieval-Augmented Generation) applications.

# Systematically Improving RAG Project Structure

This is a documentation and workshop project for RAG (Retrieval-Augmented Generation) applications.

## Key Directories
- **`docs/`** - Main documentation using MkDocs
  - **`docs/talks/`** - Industry expert talks and presentations
  - **`docs/workshops/`** - Workshop chapters and exercises
  - **`docs/office-hours/`** - Office hours summaries and FAQs
- **`latest/`** - Current cohort materials and case studies
- **`cohort_1/`** & **`cohort_2/`** - Previous cohort materials
- **`data/`** - Sample datasets and examples

## Important Files
- **[mkdocs.yml](mdc:mkdocs.yml)** - Site navigation and configuration
- **[docs/talks/AGENTS.md](mdc:docs/talks/AGENTS.md)** - Talk formatting guidelines
- **[docs/talks/index.md](mdc:docs/talks/index.md)** - Talk directory index
- **[pyproject.toml](mdc:pyproject.toml)** - Python dependencies using `uv`

## User Preferences
- Always use `uv` instead of `pip` for Python package management
- Write at 9th-grade reading level
- Use async over synchronous Python code when possible
- Match existing document styles when writing
- No emojis in documentation

## Build System
- Uses MkDocs for documentation site generation
- Deployed to GitHub Pages
- Run `mkdocs serve` for local development
- Run `mkdocs build` for production builds

---
> Source: [jxnl/systematically-improving-rag](https://github.com/jxnl/systematically-improving-rag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
