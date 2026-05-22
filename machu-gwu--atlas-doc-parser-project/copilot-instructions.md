## atlas-doc-parser-project

> See @README.rst for complete project overview.

# atlas_doc_parser Project Guide

## Project Overview

See @README.rst for complete project overview.

## Core Development Guides

## Essential Commands

- **All Operations**: @./Makefile (run `make help` for full command list)
- **Python Execution**: Use `.venv/bin/python` for all Python scripts in:
  - `debug/**/*.py` - Debug utilities
  - `scripts/**/*.py` - Automation scripts
  - `config/**/*.py` - Configuration deployment
  - `tests/**/*.py` - Unit and integration tests

## Quick Start Workflow

1. **Setup**: `make venv-create && make install-all`
2. **Update Dependencies**: ``make poetry-lock && make poetry-export && make install``
3. **Development**: Edit code in ``atlas_doc_parser/**/*.py`` → Run tests ``.venv/bin/python tests/**/*.py``
4. **Testing**: `make test` or `make cov` for coverage
5. **Build Document**: `make build-doc && make view-doc` for build sphinx docs and open local html doc site in web browser

---
> Source: [MacHu-GWU/atlas_doc_parser-project](https://github.com/MacHu-GWU/atlas_doc_parser-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
