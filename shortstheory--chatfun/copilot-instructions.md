## chatfun

> Use this repo as a small, reproducible fine-tuning pipeline.

# Repository Guidelines

## Project Structure & Module Organization
Use this repo as a small, reproducible fine-tuning pipeline.
- `data/`: raw WhatsApp exports (`part1.txt`, `part2.txt`).
- `processed/`: cleaned intermediate data (parsed turns, filtered rows).
- `datasets/`: final train/val/test JSONL used for fine-tuning.
- `scripts/`: preprocessing, dataset build, eval, and sampling scripts.
- `models/`: adapter/model metadata, run configs, and eval reports.

Keep stages separate: raw text should never be edited in place.

## Build, Test, and Development Commands
Preferred local flow (Python):
- `python -m venv .venv && source .venv/bin/activate`: create env.
- `pip install -r requirements.txt`: install dependencies.
- `python scripts/parse_whatsapp.py --in data --out processed/turns.jsonl`: parse chat logs into speaker turns.
- `python scripts/build_dataset.py --in processed/turns.jsonl --out datasets --val-ratio 0.1 --test-ratio 0.1`: create splits.
- `python scripts/train_lora_unsloth.py --train datasets/train.jsonl --val datasets/val.jsonl`: run QLoRA fine-tune.
- `python scripts/eval.py --model <model_id> --test datasets/test.jsonl`: evaluate style and coherence.

## Coding Style & Naming Conventions
- Python: 4-space indentation, type hints on public functions, small pure helpers.
- Files/functions: `snake_case`; classes: `PascalCase`; constants: `UPPER_SNAKE_CASE`.
- Prefer deterministic scripts (explicit seeds, explicit input/output paths).
- Run formatting/linting before commits (`ruff check .`, `ruff format .` if configured).

## Testing Guidelines
- Put tests in `tests/` with names `test_<module>.py`.
- Minimum coverage focus: parser correctness and dataset split integrity.
- Add fixture-based tests for WhatsApp quirks seen in this repo:
  - leading invisible mark before timestamp (`‎[... ]`)
  - multiline messages
  - system events and media placeholders
  - edited/deleted markers
- Run tests with `pytest -q`.

## Commit & Pull Request Guidelines
- Use Conventional Commits: `feat:`, `fix:`, `chore:`, `test:`, `docs:`.
- One logical change per PR (for example: “parser improvements” vs “parser + trainer refactor”).
- PR should include:
  - what changed and why
  - command(s) run (`pytest -q`, dataset build command)
  - before/after sample outputs for parser or generation changes

## Fine-Tuning Strategy Notes
- Start with one group model using explicit speaker tags (`<arnav>`, `<vinit>`, etc.).
- Exclude system messages and `Meta AI` rows from training by default.
- Move to per-speaker adapters only if style imitation is weak for specific users.

---
> Source: [shortstheory/chatfun](https://github.com/shortstheory/chatfun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
