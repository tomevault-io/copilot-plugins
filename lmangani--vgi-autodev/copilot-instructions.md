## vgi-autodev

> Guidance for AI agents (and humans) working in this repository. The README is the

# CLAUDE.md — vgi-autodev

Guidance for AI agents (and humans) working in this repository. The README is the
product doc: what the functions are and how to call them. This is the build doc:
how it fits together, and where it bites.

## What this is

A [VGI](https://query.farm/vgi/) worker exposing [Dev-0.4b](https://github.com/mpnikhil/dev-0.4b)
typed decisions to DuckDB. Dev is a 399M ModernBERT-large encoder with one universal
choice head. No API keys, no cloud calls, no per-query costs.

Three question types:
- **noul** (yes/no/unsure/likely): probability between 0 and 1
- **choice**: pick one option from a set, with confidence
- **score**: place content on an ordered scale

## Layout

One module per function, plus shared infrastructure.

```
autodev_worker.py       worker script (ATTACH via launch:…), PEP-723 self-resolving
serve.py                HTTP entry point (uv run serve.py --port 8000)
vgi_autodev/
  __init__.py           version
  worker.py             the catalog: which functions exist, catalog/schema docs
  engine.py             the ONLY module that loads the model
  is_interesting.py     scalar function for WHERE/CASE/ORDER BY
  noul.py               table function for yes/no via LATERAL
  choice.py             table function for classification via LATERAL
  score.py              table function for ordered scales via LATERAL
  meta.py               catalog-tag helpers
tests/
  test_engine.py        model inference tests
  test_end_to_end.py    real SQL via haybarn
```

## Core conventions

- **One inference path.** Every function calls `vgi_autodev.engine`, which is the only
  module that loads `dev.inference.Predictor`. VGI Arrow batches are
  length-sorted, packed to Dev's token budget, and run as autocast forwards.
- **Errors throw; only a NULL input is NULL.** An inference failure must propagate.
  A NULL answer is indistinguishable from a NULL input.
- **Validate at bind.** Missing `instructions` or `criteria` should fail when the
  query is planned, before inference runs.
- Python ≥ 3.13, `from __future__ import annotations`, Google docstrings.
- Copyright header: `# Copyright 2024 Lorenzo Mangani - https://github.com/lmangani/vgi-autodev`.

## The model

Inference is [Dev-0.4b](https://github.com/mpnikhil/dev-0.4b) (`mpnikhil/dev-0.4b`).

- **Architecture**: ModernBERT-large + single universal choice head
- **Readouts**: noul / choice / score with post-hoc temperature scaling
- **Checkpoint**: `DEV_CHECKPOINT` (default `mpnikhil/dev-0.4b`)
- **Device**: `DEV_DEVICE` (default `auto` → MPS or CUDA)
- **Batching**: Pack to `DEV_BATCH_TOKENS=32768` (32×1024 padded slots).
  Measured MPS plateau: 4.14 ms/row on title-length noul. 48k+ is slower;
  mixed lengths padded to the longest sequence are much slower. Leave
  `DEV_BATCH_SIZE` at 0 (token budget only). `DEV_AUTOCAST=1` (MPS fp16 /
  CUDA bf16) is ~15% faster than fp32.
- **Noul**: BoolQ passage verification. Topic fragments become questions; P(yes)
  is σ(s_yes − s_no) over `["No", "Yes"]` spans.
- **Choice**: MAP *values* are the listwise candidate spans; keys are VGI labels.
- **Score**: expected rubric index Σ j·P_j, divided by n−1 for VGI 0..1.
- **Singleton**: `vgi_autodev.engine.get_agent()` loads once per process

## Sharp edges

1. **First-call latency.** The first query downloads and loads the model (~20-30s).
   Subsequent queries in the same session are fast.

2. **Memory pressure.** The model needs ~2GB RAM. On memory-constrained systems,
   inference may be slow due to swapping.

3. **Long inputs.** Dev refuses to truncate; a row over `max_length` tokens
   raises rather than silently clipping.

4. **ATTACH via `launch:` for SQL throughput.** VGI's batched LATERAL operator
   (`vgi_batch_lateral`, default on) is `ParallelOperator=true`: DuckDB sends
   concurrent Arrow batches even when the worker advertises `Meta.max_workers=1`.
   Stdio `LOCATION 'uv run autodev_worker.py'` spawns one subprocess per
   substream — ~16 Dev loads on 32 threads. Use
   `LOCATION 'launch:uv run autodev_worker.py'` (one unix worker) and let
   `engine._infer_lock` serialize GPU forwards. `vgi_autodev.attach.attach_location()`
   builds the string. Scalar-only queries on stdio already batch 500 rows in one
   call (~2s); LATERAL needs `launch:`.

5. **PEP-723 headers must match pyproject.toml.** The headers in `autodev_worker.py`
   and `serve.py` are what `uv run` resolves. Keep them in sync with
   `[project.dependencies]`.

## Testing

```sh
uv run pytest -v                 # all tests
uv run pytest tests/test_engine.py  # just model tests
```

The end-to-end tests require `haybarn` (DuckDB with VGI extension). They skip
automatically if haybarn is not found.

## Running

### DuckDB attach (default)

```sql
ATTACH 'autodev' (TYPE vgi, LOCATION 'launch:uv run autodev_worker.py');
```

### HTTP server

```bash
uv run serve.py --port 9876
```

```sql
ATTACH 'autodev' (TYPE vgi, LOCATION 'http://localhost:9876');
```

### From installed package

```bash
pip install vgi-autodev
vgi-autodev        # stdio
vgi-autodev-http   # http
```

## Use cases

This worker is designed for:

- **Content filtering**: Keep articles/posts matching your interests
- **Ticket routing**: Classify support requests to the right team
- **Severity triage**: Prioritize issues by urgency
- **Dataset curation**: Filter training data by quality or relevance
- **Log analysis**: Flag anomalous or important entries
- **Research filtering**: Find papers/articles on specific topics
- **Content moderation**: Flag potentially problematic content
- **Sentiment detection**: Classify positive/negative/neutral

All inference runs locally, so it's suitable for:
- Sensitive data that can't leave your machine
- Offline environments
- Cost-sensitive batch processing
- Development and prototyping

---
> Source: [lmangani/vgi-autodev](https://github.com/lmangani/vgi-autodev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
