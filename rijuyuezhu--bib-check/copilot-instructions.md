## bib-check

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

bib-check is a CLI tool that cleans, validates, and enhances BibTeX bibliography files. It supports AI-powered corrections (via OpenAI-compatible APIs like DeepSeek), DBLP database lookups, and an interactive vim-like TUI for reviewing changes.

## Build & Development Commands

```bash
# Install dependencies (uses uv as package manager)
uv sync

# Run the tool
uv run bib-check input.bib [output.bib] [--ai] [--dblp] [--tui] [--suppress-type]

# Lint
uv run ruff check src/
uv run ruff check --fix src/   # auto-fix

# Format
uv run ruff format src/
```

Requires Python >= 3.12. No test suite exists yet.

## Architecture

The CLI entry point (`cli.py:main`) parses args into a `Config` dataclass, then either runs batch processing or launches the TUI.

**Core processing pipeline** (`converter.py:BibConverter`):
1. `parse()` — reads BibTeX via bibtexparser
2. `process_entry()` — per-entry pipeline combining: `check_dblp()` → `ai_revise_entry()` → `format_entry()`
3. `write()` — outputs formatted BibTeX

**Key modules:**
- `converter.py` — core BibTeX processing, DBLP replacement, field stripping per entry type, fzf-based interactive selection
- `ai.py` — `AIReviser` wraps OpenAI-compatible API with specialized prompts for title casing, journal names, and proceedings names
- `dblp.py` — `DblpSearch` queries DBLP API by title, returns structured results
- `tui.py` — vim-like terminal UI with raw mode key capture; supports navigation (j/k/gg/G), selection (space/V/a), and actions (d=dblp, r=revise, f=format, w=write)
- `config.py` — `Config` dataclass with `from_args()` factory
- `colors.py` — ANSI color constants

**Entry types:** Primary support for `article` and `inproceedings`, each with a defined set of required/optional fields in `converter.py`.

## Conventions

- Linting: ruff with isort-compatible import sorting (`select = ["I"]` extended)
- AI defaults: DeepSeek API (`deepseek-chat` model, `DEEPSEEK_API_KEY` env var)
- Errors log to `bib-check.log` rather than failing hard
- Typed codebase (PEP 561 `py.typed` marker)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rijuyuezhu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rijuyuezhu)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
