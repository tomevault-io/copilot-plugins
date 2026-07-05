## mac-ansible

> - Avoid adding purely explanatory code comments, but do not remove pre-existing ones.

## Ground rules

- Be concise.
- Avoid adding purely explanatory code comments, but do not remove pre-existing ones.
- Avoid code comments that highlight or describe new or old code.
- Avoid long summaries of your work after doing it.
- Files often use curly quotes (‘ ’ “ ”) — never convert them to straight quotes, and when editing files containing them, maybe use `write_file` to rewrite the whole file rather than `edit_file`, to avoid fuzzy match failures.
- In a review, list topics with numbers.

## Python

- Use `uv run python` to run Python.
- If a project has a `tox.ini` file, use `tox` to run tests, selecting just one environment by default, like `tox -e py314-django61`.

---
> Source: [adamchainz/mac-ansible](https://github.com/adamchainz/mac-ansible) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-05 -->
