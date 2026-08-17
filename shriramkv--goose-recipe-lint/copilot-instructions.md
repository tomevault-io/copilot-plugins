## goose-recipe-lint

> Guidance for AI coding agents working in this repository.

# AGENTS.md

Guidance for AI coding agents working in this repository.

## What this is

A validator for goose recipe files. It checks required fields, parameter and
extension blocks, and whether declared parameters line up with the template
variables the recipe references. Pure Python, standard library only for the core.
No network access. Read only against the recipe under test.

## Layout

- `goose_recipe_lint/loader.py` reads JSON natively and the YAML subset recipes
  use, including literal and folded block scalars. Uses PyYAML when installed.
- `goose_recipe_lint/template.py` extracts template variable references from
  `{{ ... }}` and `{% ... %}`, excluding loop and set locals.
- `goose_recipe_lint/rules.py` holds all lint rules. Add new checks here.
- `goose_recipe_lint/finding.py` is the finding and result model with scoring.
- `goose_recipe_lint/report.py` renders console, JSON, Markdown, and badge output.
- `goose_recipe_lint/cli.py` is the argparse entry point and CI gates.
- `samples/` holds a valid and a broken recipe used by tests and demos.

## Rules for changes

- Keep the core dependency free. PyYAML is an optional convenience, never required.
- Accept both the canonical parameter form (key / input_type / requirement) and
  the legacy form (name / type / required). Do not drop that tolerance.
- Template checks are heuristic. Prefer a missed edge case over a false positive:
  keep undeclared-variable at warning, not error.
- Be tolerant of unknown recipe keys. Never raise on fields you do not recognise.
- Do not use em dashes in prose or comments.
- Run `python3 -m unittest discover -s tests` before finishing.

---
> Source: [shriramkv/goose-recipe-lint](https://github.com/shriramkv/goose-recipe-lint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
