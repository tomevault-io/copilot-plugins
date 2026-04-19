## byte-size-travel

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## CLI Usage

- **Use `gh` CLI for all GitHub operations** -- issues, PRs, labels, runs, checks. Never use raw git commands when `gh` can do the same thing (e.g., `gh pr status` instead of manual branch tracking).
- **Use dedicated tools over Bash** -- Read instead of `cat`, Grep instead of `grep`/`rg`, Glob instead of `find`/`ls`. Reserve Bash for commands that have no dedicated tool equivalent (git, uv, gh).
- **Minimize redundant git commands** -- don't run exploratory git commands to "check state" when you already know the state from prior output. One `git status` is enough.
- **Always check docs with Context7** -- before implementing any feature or using any library API, use the Context7 MCP to look up the current documentation. Do not rely on memory or training data for library APIs -- always verify the correct, up-to-date usage.

## Project Context

byte-size-travel is a Python travel newsletter system that collects articles from RSS feeds and email sources, enriches them with AI-generated metadata via OpenAI, selects content for newsletters, generates newsletter HTML, and distributes via Amazon SES and a static website.

### Tech Stack

- **Python:** 3.12, managed with uv
- **Database:** SQLite (local article storage -- fetch and processed layers)
- **AI:** OpenAI (gpt-4.1-mini) for content enrichment and newsletter writing
- **Email:** Amazon SES for newsletter delivery
- **Data validation:** Pydantic
- **Parsing:** feedparser + BeautifulSoup for RSS/HTML, imaplib for Gmail IMAP
- **Templates:** Handlebars (Node.js) for newsletter HTML generation
- **Website:** Static HTML/CSS/JS deployed to S3

## Commands

All commands run from the project root.

```bash
# install dependencies
uv sync

# lint
uv run ruff check .

# auto-fix lint issues
uv run ruff check --fix .

# format code
uv run ruff format .

# run tests
uv run pytest tests/

# run the newsletter pipeline
PYTHONPATH=src uv run python src/main.py
```

## Principles

- **Simplicity and best practices** -- these are our guiding values. Every decision should favor the simplest correct solution that follows established conventions.
- **Always follow conventional best practices** -- for Python and every library we use. Research the current recommended approach before proposing anything. Never suggest a non-standard pattern without explicitly calling it out and justifying why. Before implementing any feature or using any library API, use Context7 to look up the current documentation and verify the correct, up-to-date usage.
- **No shortcuts or "good enough" defaults** -- if there's an established convention, use it. Don't invent project structure, naming, or patterns when a well-known standard exists. If you know the right way to do something, do it that way the first time.
- **Fix problems properly, don't suppress them** -- when a linter, type checker, or test flags an issue, fix the underlying code. Don't silence warnings with ignores, suppressions, or config overrides unless the rule is genuinely inapplicable. When using `noqa`, always document the reason in the comment (e.g., `# noqa: ARG001 -- required by framework`). A bare `noqa` without justification is not acceptable.
- **Never use falsy checks for numeric values** -- `value or None` and `if not value` silently convert `0`, `0.0`, and `""` to `None`/`False`. Use explicit checks: `value is not None`.
- **Never ignore errors** -- if a command fails, stop and investigate. Don't skip past errors, don't assume they're harmless, and don't apply hacky workarounds to make them go away. Diagnose the root cause first, then fix it properly.
- **Own all test failures** -- when a test fails during your work, fix it. Don't dismiss failures as "pre-existing" or "not caused by my changes." If the fix is genuinely complex and unrelated, file it as a GitHub issue, but try to fix it first.
- **Fix deprecation warnings immediately** -- when tests or commands emit deprecation warnings from our code, fix them right away. Warnings from third-party libraries are out of our control, but anything in our code should use the current non-deprecated API.
- **Greenfield discipline** -- don't leave unused columns, dead code, placeholder features, or "just in case" abstractions. If something isn't needed now, remove it. Every line of code should serve a current purpose.
- **Open issues for necessary but out-of-scope work** -- if implementing a feature reveals work that's needed elsewhere, open a GitHub issue with appropriate labels instead of leaving it undocumented. Necessary follow-up work must never silently fall through the cracks.
- **No ephemeral docs in version control** -- planning documents, implementation plans, and design notes are temporary artifacts. Don't commit them. The git history and PR descriptions capture the "why" permanently.
- **Dependencies** -- use `uv add <package>` to add dependencies. uv resolves the latest version and pins it automatically. Never manually edit `pyproject.toml` for dependencies, and never use `uv pip install` or touch `uv.lock` directly -- `uv.lock` is always managed by uv commands (`uv sync`, `uv add`, `uv lock`). When resolving a `uv.lock` merge conflict, run `uv sync` to regenerate it. Adding a new dependency is a significant decision -- always consult the user first.
- **Testing** -- use `pytest`. Never use inline `python -c` scripts to verify behavior -- always write a proper test file and run it with pytest.

## Code Style

- Ruff with `select = ["ALL"]` -- nearly every rule enabled
- Ignored rule categories: `D` (docstrings), `FIX`/`TD` (TODOs), `CPY` (copyright)
- Type annotations are required
- No commented-out code -- use git history instead
- No emojis anywhere -- not in code, comments, commits, PRs, or output
- Python 3.12, line length 88, double quotes
- Catch specific exceptions, not bare `Exception` -- ruff `BLE001` is enabled
- PR descriptions are concise -- short summary bullets and a "Tests" section listing what was verified
- GitHub Actions: never interpolate user inputs into shell -- pass inputs via `env:` and reference as `$ENV_VAR`

## Workflow

- **Simple git only** -- never amend, squash, rebase, or force-push. Always make a new commit. Keep the git toolkit to `add`, `commit`, `push`, `pull`, `branch`, `checkout`, `merge`, `status`, `diff`, `log`.
- **Small, structured changes only** -- never do multiple steps at once. One logical change per step, verifiable and testable before moving on.
- Lint and format after every change: `uv run ruff check --fix . && uv run ruff format .`
- Run tests after every change to confirm nothing broke.
- If a task requires multiple changes, break it into the smallest increments that can each be verified independently.
- **Use `/verify` for validation** -- lint, format, and test in one step.
- **Use `/ship` for commits, branches, and PRs** -- never manually create branches, commit, push, or open PRs. When work is ready, use the `/ship` command which handles the full workflow.
- **Label issues properly** -- every issue must have type, area, and priority labels. Run `gh label list` to see available labels and only use those.

## Comment Style

- Comments are simple lowercase descriptions -- no numbers, no prefixes, no extra symbols
- This applies everywhere: SQL, Python, and any other language in the project
- Example: `# validate article exists` not `# 1. validate article exists`

## Test Organization

- **Shared setup belongs in `conftest.py` from the start** -- if a lookup or resource will be used by more than one test file, put it in `conftest.py` as a fixture immediately.
- **Single-file helpers stay local** -- only extract to conftest when 2+ files share the pattern.
- **Session-scoped fixtures for stable data** -- data that doesn't change between tests should be session-scoped.
- **Test files mirror src layout** -- keep tests organized to match the source structure.

## Project Structure

```
src/                              # all application code
  main.py                        # pipeline entry point
  config/                        # configuration and logging
    source_manager.py            # RSS/email source YAML config management
    logging_config.py            # logger setup with file rotation
  content/                       # content pipeline
    fetching/                    # RSS and email feed fetching
      parsers.py                 # feed parsing (RSS, Gmail IMAP, HTML cleaning)
      rss_full_fetch.py          # full content fetch for partial RSS entries
    enriching/                   # AI-powered content enrichment
      article_enricher.py        # OpenAI-based article metadata extraction
    selection/                   # newsletter content selection
      article_selector.py        # scoring and selection algorithm
    writing/                     # newsletter generation
      newsletter_writer.py       # OpenAI-based newsletter HTML generation
  database/                      # SQLite database layer
    fetch_database.py            # raw article storage
    processed_database.py        # enriched article storage
    populate_db.py               # source-to-database pipeline
  models/                        # Pydantic models
    schemas.py                   # ProcessedArticle, DealData, etc.
  services/                      # external service clients
    openai/                      # OpenAI API client
    amazon_ses/                  # Amazon SES email client
tests/                           # mirrors src layout
config/                          # YAML source configuration
data/                            # SQLite databases and content JSON
website/                         # static website (deployed to S3)
html_templates/                  # email newsletter HTML templates
tools/                           # Node.js helper scripts (Handlebars rendering)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JohnathanOneal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
