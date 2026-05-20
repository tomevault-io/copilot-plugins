## scans2any

> This document provides context, architectural guidelines, and coding standards

# AI Agent Instructions for scans2any

This document provides context, architectural guidelines, and coding standards
for AI coding assistants working on the `scans2any` repository.

## Project Overview

`scans2any` is a tool designed to convert infrastructure scans (like Nmap,
Nessus, Aquatone, etc.) into various output formats such as Markdown tables,
YAML, Typst, HTML, CSV, and more.

## Tech Stack & Tooling

- **Language:** Python >= 3.12
- **Package & Project Manager:** `uv` -- never use `pip`, `poetry`.
- **Build System:** `hatchling`
- **Key Dependencies:** `python-libnmap`, `pandas`, `pyyaml`, `tabulate`,
  `jinja2`, etc.

## Code Style & Linting

The project uses `ruff` for both formatting and linting.

- **Commands:** Run `ruff format` and `ruff check` before committing.
- **Key Rules (from `ruff.toml`):**
  - **No `print` statements:** Use the `logging` module instead (`T` rule).
  - **Naming:** Follow PEP8 naming conventions (`N` rule).
  - **Imports:** Keep imports organized (`I` rule).
  - **Line Length:** `E501` is ignored, but try to keep lines reasonably long.

Run `uv ty check` for type checking before committing.

## Architecture & Internals

The core workflow of `scans2any` is divided into three main phases:

### 1. Parse & Combine

Input files are parsed using their respective parsers (e.g., `nmap_parser.py`,
`aquatone_parser.py`), resulting in an `Infrastructure` object for each file.
These objects are then unioned.

- **Optimized Combination:** The project uses an optimized union-find (disjoint
  set) clustering across all hosts based on shared IP addresses OR hostnames.
  Any hosts linked transitively through these tokens are merged exactly once.

### 2. Merge & Filter

Collisions in the combined infrastructure (e.g., multiple detected service names
or banners for the same port) are resolved, and filters are applied to compress
information (e.g., filtering blacklisted IPs or banner info).

- **Collision Resolution Priority:**
  1. Application-internal auto-merge rules (e.g., resolve `https`+`www` to
     `https`)
  2. User-defined auto-merge rules (in merge-file)
  3. User-defined manual merge rules (in merge-file)

### 3. Check & Output

If unresolved collisions remain, a merge-file is created, and the user is
prompted to resolve them manually. Otherwise, the output is generated in the
chosen format.

## Testing Guidelines

- **Framework:** `pytest`
- **Structure:** Tests are organized in the `tests/` directory, which includes:
  - `unit_tests/`
  - `integration_tests/`
  - `performance_tests/`
  - `smoke_tests/`
- **Running Tests:** Use `uv run pytest` to run the test suite. Ensure new
  features or bug fixes are accompanied by appropriate tests in the relevant
  directories.

### Performance

New changes should not be slower. Use `tests/performance_tests` to run
performance tests and ensure no regressions are introduced. Compare to
`*results.json` or `results.txt` files in the `tests/performance_tests`
directory to verify performance benchmarks. Test custom cases with `hyperfine
--warmup 1` to ensure no regressions in speed.

## Miscellaneous

- Never create new MD files witout approval. Always check with the team before
  adding new documentation files.
- Always update the zsh completion file when adding new CLI options or commands.
- Use the `CHANGELOG.md` file to document all significant changes, new features,
  and bug fixes. Follow the format used in previous entries for consistency.

---
> Source: [softScheck/scans2any](https://github.com/softScheck/scans2any) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
