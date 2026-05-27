## experiment-code-template

> This project uses [`AGENTS.md`](../AGENTS.md) as the single source of truth.

# GitHub Copilot Instructions

This project uses [`AGENTS.md`](../AGENTS.md) as the single source of truth.
Read it before suggesting code.

## Hard rules

- Methods, baselines, trainers, experiments are registered via
  `@register_method`, `@register_baseline`, `@register_trainer`, `@register_exp`
  (all in `expkit.core.registry`). Don't invent new decorators.
- Paper-agnostic code lives in `expkit/` (14 subpackages: `core`, `configs`,
  `utils`, `data`, `models`, `trainers`, `metrics`, `inference`, `reporting`,
  `callbacks`, `profiling`, `interpret`, `hyperopt`, `domains`).
- Paper-specific code lives in `projects/<slug>/` (method/, baselines/,
  experiments/, configs/, paper/, notes/). Never modify `expkit/` for one paper.
- Hyperparameters live in Hydra `configs/`, not in code.
- Every run writes three files into `experiments/<exp>/results/`:
  `metrics.json`, `env.json`, `run_signature.json`. Never skip any of them.
- Use `expkit.utils.*` instead of reinventing logging / hashing / git / seed.

## Commands

- Init: `python -m expkit init <name> --from <paradigm>`
- New exp: `python -m expkit new-exp --project <p> --name <n>`
- New baseline: `python -m expkit new-baseline --project <p> --name <n>`
- Run: `python -m expkit run project=<p> exp=exp_NNN_<name>` (Hydra-style)
- Aggregate: `python -m expkit reporting-table --project <p> --metric <m>`
- Pre-flight: `python -m expkit doctor`

There is **no Makefile** and **no `scripts/scaffold_from_paper.py`** — all
operations are subcommands of `python -m expkit`.

---
> Source: [qoham/experiment_code_template](https://github.com/qoham/experiment_code_template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
