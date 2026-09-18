## ducheck

> - **Package manager:** `uv` — use `uv run`, `uv add`, `uv sync`, `uv lock`. Never use `pip` directly.

# Ducheck — Agent Instructions

## Stack & Toolchain

- **Package manager:** `uv` — use `uv run`, `uv add`, `uv sync`, `uv lock`. Never use `pip` directly.
- **Python:** 3.13 (pinned in `.python-version`).

## Project Purpose

Check English LaTeX documents for:
- Spelling (`hunspell`)
- Grammar (`language-tool-python`)
- Abbreviation consistency (defined-before-use, conflicting definitions, casing)
- Terminology consistency (hyphenation, casing)
- Data format consistency (spacing, units, dashes, etc.)

## Design Principles

- Lightweight — prefer standard library or mature, small-dependency libraries.
- Easy to maintain — keep the codebase simple and flat while it remains small.
- Sustainable — free, well-maintained dependencies only.
- LaTeX-aware — use `pylatexenc` for parsing, handle `\include`-ed files and vector graphics (PDF/EPS) for text extraction.

## Commands

```bash
uv run ducheck --help       # list CLI options
uv run ducheck --tui        # choose parameters interactively
uv add <package>            # add a dependency
uv add --dev <package>      # add a dev dependency
uv sync                     # sync venv with pyproject.toml
uv lock                     # regenerate lockfile
uv run ruff check .         # lint
uv run ruff format .        # format
uv run ruff check --fix .   # auto-fix lint issues
```

## CLI

```bash
uv run ducheck <entry.tex>             # extract text from a LaTeX document
uv run ducheck <entry.tex> -o out.txt  # write output to file
```

Output format uses `%%% BEGIN <path>` / `%%% END <path>` markers to delimit per-file extracted text.

## Distribution and Frontends

- Standalone builds currently target macOS 15+ on Apple Silicon only.
- Build: `uv run --group build python scripts/build_release.py`.
- Verify: `uv run python scripts/smoke_release.py dist/ducheck/ducheck`.
- `checking.py` owns shared checking orchestration; frontends own arguments and presentation.
- `diagnostics.py` normalizes checker results once; all frontends consume the shared diagnostic list.
- The TUI is opt-in with `--tui` and reuses `prompt_toolkit`.
- Keep TUI pages inside one Application; reuse the shared review view and text reporting helpers.
- Reference web search discovers publication pages; never treat search snippets as verified metadata.
- User configuration is TOML in platformdirs' user configuration directory; do not load project `.env` files.
- Keep user documentation in English and developer setup separate from release installation.

---
> Source: [Thysrael/ducheck](https://github.com/Thysrael/ducheck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
