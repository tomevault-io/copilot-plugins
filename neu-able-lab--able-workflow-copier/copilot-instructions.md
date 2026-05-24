## able-workflow-copier

> This repository contains a Copier template for creating a Snakemake workflow and an associated python package.

# Agent Contribution Guide

This repository contains a Copier template for creating a Snakemake workflow and an associated python package.

## Environment

The setup script `AGENTS-setup.sh` should have already been run, setting up the environment as follows:

- The `.bashrc` file has been modified to activate the conda environment `able-workflow-copier` that include all the development dependencies.
- `tox` is available for environment isolation for testing
- Example templates from `example-answers/` are rendered in the `sandbox/`

## Documentation

The mkdocs documentation for the template resides in the `docs/docs/` directory. Additionally, many directories contain `README.md` files.

## Testing

Tox is used to run all the tests. The environment initialization already installed all the packages. To speed things up, run tox in parallel and skip the rule integration environments:

```bash
tox run-parallel --parallel auto --parallel-no-spinner --skip-pkg-install
```

See `tox.ini` for the configuration and `sandbox/able-workflow-copier-dev/docs/docs/contributing/testing.md` for more details on testing different parts of the code.

---
> Source: [NEU-ABLE-LAB/able-workflow-copier](https://github.com/NEU-ABLE-LAB/able-workflow-copier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-24 -->
