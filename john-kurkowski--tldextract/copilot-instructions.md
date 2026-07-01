## tldextract

> See [README.md](README.md) for contributor setup.

# Repository Guidance

See [README.md](README.md) for contributor setup.

## Tests

While iterating on a change, prefer the smallest test invocation that exercises
the behavior you are working on, such as a specific test file or pytest `-k`
selector:

```zsh
tox -e py310 -- tests/cli_test.py
tox -e py310 -- -k private
```

Before wrapping up any code change, always run the required validation baseline:

```zsh
tox -e py310         # Fastest full pytest run while iterating
tox -e lint          # Lint (ruff check)
tox -e typecheck     # Type-check (mypy)
ruff format .        # Auto-format code
```

Run broader validation when the scope warrants it:

```zsh
tox --parallel       # Run everything: pytest on all Pythons + codestyle + lint + typecheck
```

The `py*` and `pypy*` environments forward to `pytest`; `codestyle`, `lint`, and
`typecheck` do not.

---
> Source: [john-kurkowski/tldextract](https://github.com/john-kurkowski/tldextract) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
