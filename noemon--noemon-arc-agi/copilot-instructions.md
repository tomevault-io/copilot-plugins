## noemon-arc-agi

> This repository packages the Noemon ARC-AGI 2 solver on top of the ARC benchmarking framework.

# AGENTS.md

This repository packages the Noemon ARC-AGI 2 solver on top of the ARC benchmarking framework.

Use `.venv/bin/python` for all Python commands.

## What matters most

- The Noemon solver lives in `src/arc_agi_benchmarking/noemon/`.
- The production model config is `gemini-3-1-pro-noemon-batch` in `src/arc_agi_benchmarking/models.yml`.
- The Noemon provider is batch-only: `src/arc_agi_benchmarking/adapters/noemon_gemini_batch.py`.
- Single-task entrypoint: `main.py`
- Batch entrypoint: `cli/run_all.py`
- Batch orchestration/checkpointing: `src/arc_agi_benchmarking/noemon/batch_engine.py` and `src/arc_agi_benchmarking/noemon/batch_loop.py`
- Prompt construction: `src/arc_agi_benchmarking/noemon/prompts.py` and `src/arc_agi_benchmarking/noemon/prompt_bundle.md`

## Repository shape

```text
main.py
cli/run_all.py
src/arc_agi_benchmarking/
  adapters/
  noemon/
  scoring/
  tests/
  utils/
data/
assets/
scripts/
provider_config.yml
```

## Noemon workflow

- The `noemon-gemini-batch` adapter does not support synchronous predictions.
- `main.py` routes that config into the resident batch loop instead of calling `make_prediction(...)`.
- `cli/run_all.py` is the main way to run the full Noemon solver over many tasks.
- Checkpoints are stored under `<save_submission_dir>/.noemon_checkpoints/`.
- Batch jobs are resumed from checkpoint state and registry manifests inside `.noemon_checkpoints/_batch_jobs/`.
- A finished submission is written to `<save_submission_dir>/<task_id>.json`.

## Useful commands

```bash
make install
make test
make test-verbose
make run-sample
make run-batch
make score
```

Equivalent direct commands:

```bash
.venv/bin/python -m pytest -q
.venv/bin/python main.py --data_dir data/sample/tasks --config random-baseline --task_id 66e6c45b --save_submission_dir submissions/test
.venv/bin/python cli/run_all.py --data_dir data/sample/tasks --config random-baseline --save_submission_dir submissions/test
.venv/bin/python src/arc_agi_benchmarking/scoring/scoring.py --task_dir data/sample/tasks --submission_dir submissions/test
```

Noemon sample run:

```bash
.venv/bin/python cli/run_all.py \
  --task_list_file data/evaluation.txt \
  --data_dir data/v2/arc-agi_evaluation_challenges.json \
  --config gemini-3-1-pro-noemon-batch \
  --save_submission_dir submissions \
  --num_attempts 2
```

## Testing guidance

- Run targeted tests first when changing Noemon logic:
  - `src/arc_agi_benchmarking/tests/test_noemon_batch.py`
  - `src/arc_agi_benchmarking/tests/test_noemon_prompts.py`
  - `src/arc_agi_benchmarking/tests/test_noemon_runtime_config.py`
  - `src/arc_agi_benchmarking/tests/test_main_noemon.py`
- Use `.venv/bin/python -m pytest ...`, not bare `python` or system `pytest`.
- Many tests use fake providers; if you change the real batch adapter interface, add coverage that exercises `NoemonGeminiBatchAdapter` behavior too.

## Data/task notes

- `--data_dir` accepts either a directory of per-task JSON files or a bundled JSON file keyed by task id.
- ARC tasks use `train` and `test` arrays of grids; grid values must be integers `0..9`.
- Sample tasks live in `data/sample/tasks/`.

## Environment

Common env vars:

- `GOOGLE_API_KEY` for Gemini / Noemon batch runs
- `OPENAI_API_KEY`
- `ANTHROPIC_API_KEY`
- `XAI_API_KEY`
- `FIREWORKS_API_KEY`
- `GROQ_API_KEY`
- `OPENROUTER_API_KEY`
- `HUGGING_FACE_API_KEY`

## Code conventions

- Prefer `python -m pytest` through `.venv/bin/python`.
- Keep Noemon prompt/response contracts aligned with `response_schemas.py`.
- Preserve checkpoint compatibility when changing `TaskTestRunState`, batch manifests, or provider response serialization.
- Submissions are stored as JSON with `attempt_1`, `attempt_2`, etc. for each test pair.

---
> Source: [noemon/noemon-arc-agi](https://github.com/noemon/noemon-arc-agi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
