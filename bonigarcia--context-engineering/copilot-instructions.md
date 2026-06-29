## context-engineering

> This repository implements a command-line task tracker in Python.

# Project guidance for AI coding agents

## Project purpose
This repository implements a command-line task tracker in Python.

## Working rules
- Make the smallest change that satisfies the requested behavior.
- Preserve existing public function names unless the task explicitly requires a breaking change.
- Do not edit generated files under `dist/`.
- Do not add production dependencies without explicit user approval.

## Code style
- Use Python 3.12.
- Add type hints to new public functions.
- Use `pathlib` for file-system paths.
- Follow the style already used in nearby files.

## Validation
Before reporting completion:
1. Run `python -m pytest`.
2. Run `ruff check .`.
3. Mention any command that could not be run.

## Response format
When reporting changes:
- Summarize the change in one paragraph.
- List the validation commands and their results.
- Call out risks, limitations, and files that may need manual review.

---
> Source: [bonigarcia/context-engineering](https://github.com/bonigarcia/context-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
