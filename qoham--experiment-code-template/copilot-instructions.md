## project

> Project-wide rules. Read AGENTS.md first.


# expkit Project Rules

This project uses [`AGENTS.md`](../../AGENTS.md) as the **single source of truth**.
Always read AGENTS.md before acting.

## Hard rules

- Execution unit is `Task` (`expkit/core/task.py`). Register with `@register_task("name")`.
- Compose Tasks via `Pipeline` (`expkit/core/pipeline.py`). Do not invent new orchestration layers.
- All hyperparameters live in `configs/`. Code reads config, never hardcodes.
- Outputs go to `projects/<paper>/runs/<date>/<time>/` (Hydra working dir,
  immutable) and `projects/<paper>/experiments/<exp>/results/` (canonical
  metrics view). Never overwrite — each invocation creates a new run dir.
- Every run writes `metrics.json`, `env.json`, and `run_signature.json` into
  `results/` (handled by `expkit.core.bootstrap.bootstrap_run`, invoked from
  each `projects/<slug>/experiments/exp_NNN_<name>/run.py`).
- Use `expkit.utils.*` instead of reinventing logging/cache/hashing/git/checkpoint.
- Optional deps are lazy: domain code lives behind `try/except ImportError`.

---
> Source: [qoham/experiment_code_template](https://github.com/qoham/experiment_code_template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
