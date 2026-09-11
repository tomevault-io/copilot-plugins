## tatbot-legacy

> This repository is public and is the reviewed release destination for Tatbot

# Public Tatbot repository contract

This repository is public and is the reviewed release destination for Tatbot
technical work. It is not a mirror of private development.

## Allowed content

- public-safe source code and examples;
- reproducible developer and artist documentation;
- released behavior, schemas, fixtures, and tests;
- assets whose licenses and provenance permit redistribution.

## Never publish

Do not commit private repository names or paths, host topology, addresses,
credentials, internal contacts, unreleased plans, private datasets, operator
evidence, or material implying authorization for human use.

## Checks

```bash
uv run pytest -q
./scripts/lint_code.sh
uv run sphinx-build -b html -E --keep-going -W docs docs/_build
```

The Pages workflow additionally runs linkcheck and the public disclosure scan.
Warnings, unexplained broken links, and secret findings are release blockers.

## Documentation

Start at [docs/index.md](docs/index.md). Keep the Sphinx toctree explicit; do
not use globs that can pull future or private files into the public site.

## Safety

Prefer simulation, replay, and no-arm tests. Physical integration requires an
instrumented non-human fixture, an operator-controlled stop, and an explicit
acceptance record outside this public repository.

---
> Source: [hu-po/tatbot-legacy](https://github.com/hu-po/tatbot-legacy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
