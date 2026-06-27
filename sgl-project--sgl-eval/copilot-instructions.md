## sgl-eval

> One-click accuracy evaluation harness for SGLang. Point at any

# sgl-eval — project context for Claude Code

One-click accuracy evaluation harness for SGLang. Point at any
OpenAI-compatible endpoint, get reproducible numbers.

## Core architectural principle

**Anything that decides a score is vendored verbatim from
[NVIDIA/NeMo-Skills](https://github.com/NVIDIA/NeMo-Skills).** sgl-eval
contributes only transport (OpenAI client, threadpool runner, CLI) and
the thin glue that wires upstream pieces into one command.

Enforced by:

- `sgl_eval/_vendored/nemo_skills/` holds the vendored slice.
- `sgl_eval/_vendored/nemo_skills/SOURCES.yaml` pins the upstream commit
  and records every file's source path.
- `scripts/audit_vendored.py` (run by `tests/test_vendor_audit.py`) fails
  if any vendored file still has unrewritten `from nemo_skills.X` imports.
- NS's own `tests/test_math_equal.py` + `tests/test_base_metrics.py` are
  vendored alongside and run on every `pytest`. Drift surfaces
  immediately.

## File layout

```
sgl_eval/
├── cli.py / sampler.py / runner.py / registry.py / metrics.py / types.py
│   sgl-eval's own code: transport + plumbing.
├── evals/
│   per-benchmark glue (math runner, mcq runner, registry table,
│   prompt render, prediction dict adapter, dataset loader).
└── _vendored/nemo_skills/        # DO NOT hand-edit.
    ├── SOURCES.yaml              # the manifest
    ├── math_grader.py            # extract_answer + math_equal
    ├── _metrics_base.py          # pass@k / majority@k / std-SEM
    ├── math_metrics.py           # MathMetrics (used by both math + mcq)
    ├── evaluator/{base,math,mcq}.py
    ├── dataset/<bench>/{__init__.py, prepare.py | test.txt}
    ├── prompts/*.yaml
    └── tests/                    # NS's own behavior tests
```

## Editing rules

- **Never hand-edit anything under `sgl_eval/_vendored/`.** To change
  vendored content, edit `SOURCES.yaml` and run
  `python scripts/sync_vendored.py`.
- New functionality that touches scoring (grader, aggregator, prompt,
  dataset prep) goes through vendoring. New transport / runner / CLI
  features are SE code, fine to add directly.
- Sampling / generation defaults belong in
  `sgl_eval/evals/_registry.py:_TABLE`, not in vendored code.

## Defaults

Sampling params (`temperature=0.0`, `top_p=0.95`, `max_tokens=None`)
match upstream NeMo-Skills' `InferenceConfig`. They are
**model-dependent**; users override via CLI per-model
(`--temperature 1.0` for DSv3.2/V4, `0.6` for R1, etc.). The runner
warns when `n_repeats > 1` and `temperature == 0.0`.

Per-benchmark `default_n_repeats` and `thinking` are sgl-eval's choice
(NS leaves both to CLI). They live in `_registry.py:_TABLE`.

## Testing

```bash
pytest                                # ours + vendored NS corner cases
python scripts/audit_vendored.py      # vendor import sanity
pre-commit run --all-files            # lint + format + codespell
```

## Available skills

- `/add-benchmark <name>` — vendor a new dataset module + register it.
- `/vendor-update` — bump the synced NeMo-Skills SHA + verify.
- `/review-vendor-coverage` — audit whether SE code creeps into scoring.

---
> Source: [sgl-project/sgl-eval](https://github.com/sgl-project/sgl-eval) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
