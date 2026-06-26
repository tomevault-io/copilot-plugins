## jarify

> Guidance for AI agents working in this repository.

# AGENTS.md

Guidance for AI agents working in this repository.

## What this project is

Jarify is a SQL formatter and linter for DuckDB, built on [sqlglot](https://github.com/tobymao/sqlglot). It parses SQL with the DuckDB dialect and rewrites it through an opinionated, non-configurable style.

## Dev setup

Requires [mise](https://mise.jdx.dev) and [uv](https://docs.astral.sh/uv/).

```bash
uv sync --all-groups
```

## Common tasks

| Task | Command |
|------|---------|
| Run tests | `mise run test` |
| Run linter | `mise run lint` |
| Run both | `mise run check` |
| Format source | `mise run fmt` |
| Regenerate rule docs | `mise run gen-rules-doc` |

**Always run `mise run check` before committing** — this runs both lint (`ruff check`) and tests. Never push without a clean check. CI will fail if you do.

## Adding or changing a rule

Load `.agents/skills/add-formatter-rule.md` before implementing a new or changed formatting rule — it covers the full workflow including sqlglot default auditing, fixture generation, and style-guide updates.

1. **Formatting rules** live in `src/jarify/generator.py` (override a generator method) or `src/jarify/rules/` (subclass `FormatterRule` and implement `apply()`).
2. **Lint-only rules** live in `src/jarify/rules/` (implement `check()`, leave `apply()` as a no-op).
3. Register new rules in `src/jarify/rules/__init__.py` and add a config knob to `src/jarify/config.py`.
4. **Run `mise run gen-rules-doc`** to regenerate `docs/rules.md` from `RULE_CATALOG`. CI enforces this with `mise run check-rules-doc` — the step fails if the generated file is out of sync.
5. Add a fixture pair in `tests/fixtures/<category>/` and unit tests in `tests/`.
6. **Document every new or changed rule in [`docs/sql-style-guide.md`](docs/sql-style-guide.md)** with a bad/good SQL example.

## Adding fixture tests

```bash
# 1. Create the input file
echo "SELECT a, b FROM t" > tests/fixtures/<category>/<name>.input.sql

# 2. Generate the expected output
uv run pytest tests/test_fixtures.py --update-fixtures

# 3. Review the generated .expected.sql, then commit both files
```

---
> Source: [amfaro/jarify](https://github.com/amfaro/jarify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
