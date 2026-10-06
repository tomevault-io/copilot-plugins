## mopidy-spotify

> - Prefer dependencies and minimum versions available in Debian stable

# Mopidy conventions

- Prefer dependencies and minimum versions available in Debian stable
  (currently Debian 13, with Python 3.13). Check availability before adding or
  raising a dependency; explain exceptions in the pull request. Keep code
  compatible with the minimum Python version in `pyproject.toml`.
- Use Google-style Markdown docstrings, single backticks for code references,
  and comments that explain constraints or rationale.
- Mirror source packages under `tests/`, including `oauth/` and `_ext/`.
- When adapting third-party code or tests, record the upstream version and
  source, retain attribution and the license, and include the license in
  distributions. Document adaptations and keep their tests deterministic.

---
> Source: [mopidy/mopidy-spotify](https://github.com/mopidy/mopidy-spotify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
