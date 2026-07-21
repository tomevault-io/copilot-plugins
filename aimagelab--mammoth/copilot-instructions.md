## mammoth

> Terminal-agent manual for `mammoth`.

# AGENTS.md

Terminal-agent manual for `mammoth`.

This file is the canonical guide for coding agents in this repository.

## 1) Scope and priorities

- Scope: entire repo.
- Language: Python (`>=3.10`).
- Entrypoint: `main.py`.
- Highest-priority area: `models/`.
- Then: `datasets/`, then `backbone/`.
- Primary objective: preserve continual-learning correctness and reproducibility.
- Secondary objective: keep diffs small and targeted.

### Quick do/don't (models)

Do:

- edit `models/` first for method behavior changes,
- keep `observe` signature and return contract stable,
- run model-focused tests before broad suites.

Don't:

- bypass `meta_*` wrappers in training flow,
- change parser/config behavior without updating related tests,
- refactor unrelated files in the same change.

### Quick do/don't (datasets)

Do:

- keep train loader output as `(augmented, label, non_augmented)`,
- use `store_masked_loaders(...)` unless a setting truly requires custom splitting,
- validate task/mask behavior after any split/config change.

Don't:

- manually bypass `MammothDatasetWrapper` behavior for standard continual datasets,
- break `custom_task_order`/`custom_class_order` and permutation interactions,
- change dataset defaults/config semantics without parser tests.

### Quick do/don't (backbones)

Do:

- register new backbones with `@register_backbone`,
- keep constructor signature parser-friendly (dynamic args are inferred),
- preserve `forward(..., returnt=...)` compatibility used by existing models.

Don't:

- introduce required ctor args without exposing parseable arguments,
- change feature/logit output conventions without auditing dependent models,
- rely on only one `returnt` variant if models need `out/features/both/full`.

## 2) Repo intent

Mammoth is a continual-learning framework used to:

- run many CL methods across many datasets,
- support multiple settings (`class-il`, `domain-il`, `task-il`, `general-continual`, `cssl`, `biased-class-il`),
- make it easy to add models, datasets, and backbones.

## 3) Environment and canonical commands

Prefer `uv`.

Setup:

1. `pip install uv`
2. `uv sync --group dev`
3. Run everything with `uv run <command>`

Sanity commands:

- `uv run python main.py --help`
- `uv run pip install -e .`
- `uv run python -m build`

Tests:

- Full suite: `uv run pytest`
- Fast smoke: `uv run pytest tests/test_import.py tests/test_basic_functionality.py`
- Single file: `uv run pytest tests/test_er_example.py`
- Pattern: `uv run pytest -k "checkpoint and not wandb"`

Pytest options used in this repo:

- `--force_device=<id>` sets `MAMMOTH_DEVICE` and `CUDA_VISIBLE_DEVICES`.
- `--include_dataset_reload` enables destructive/slow dataset re-download tests.

Formatting/linting:

- `uv run autopep8 --recursive --in-place --aggressive --max-line-length=200 --ignore=E402 .`
- `uv run ruff check .`
- `uv run mypy .`

Docs:

- `cd docs && uv run ../utils/args.py && uv run sphinx-build -j auto . ../_build`
- or `cd docs && make html`

## 4) Where to edit

- `main.py`: parser pipeline, config merge, initialization.
- `models/`: methods.
- `models/utils/continual_model.py`: base model contract and wrappers.
- `models/utils/future_model.py`: required for `--eval_future`.
- `models/config/*.yaml`: model `default`/`best` configs.
- `datasets/`: dataset definitions.
- `datasets/utils/continual_dataset.py`: dataset contract + `store_masked_loaders`.
- `datasets/configs/<dataset>/*.yaml`: dataset configs.
- `backbone/`: backbone implementations + registration.
- `utils/args.py`: dynamic parser behavior.
- `utils/training.py`: training loop and `meta_*` calls.
- `utils/evaluate.py`: evaluation semantics and compatibility consequences.
- `tests/`: validation.

## 5) Runtime flow (debug map)

Core execution path:

1. `parse_args()` in `main.py` builds parser in stages.
2. Model parser and config files are loaded.
3. Dynamic dataset/backbone args are injected.
4. `initialize()` builds dataset -> backbone -> model.
5. `train()` in `utils/training.py` runs task loop using:
   - `model.meta_begin_task(dataset)`
   - `model.meta_observe(...)`
   - `model.meta_end_task(dataset)`

When behavior looks wrong, inspect `meta_*` wrapper behavior first.

## 6) Models (most important)

### Discovery

- `models/__init__.py` supports both:
  - explicit registry (`register_model`, `REGISTERED_MODELS`), and
  - legacy autodiscovery from `models/*.py` via `ContinualModel` subclass detection.
- Name matching tolerates `_` and `-` differences.

### Required model contract

Every model must:

- inherit `models.utils.continual_model.ContinualModel`,
- define `NAME` and `COMPATIBILITY`,
- implement `observe(self, inputs, labels, not_aug_inputs, epoch=None, ...)`.

`observe` can return either:

- scalar loss, or
- dict with at least `loss`.

`forward` nuance:

- If `'class-il'` and `'general-continual'` are both missing from `COMPATIBILITY`, evaluation calls `model(inputs, task_id)`.
- In that case, ensure `forward` supports a task label argument.

### Wrapper semantics

`meta_observe` handles critical framework behavior:

- removes unlabeled samples (`label == -1`) for models without `'cssl'` compatibility,
- filters extra kwargs to match `observe` signature,
- asserts required observe args are present,
- handles wandb autologging.

`meta_begin_task` updates class/task counters and optimizer reload logic.

Do not bypass wrappers in training code.

### Compatibility and future eval

- `COMPATIBILITY` must reflect supported settings accurately.
- There is no single hard assert for dataset-setting compatibility; mismatches can degrade evaluation meaningfully.
- `--eval_future=1` requires a `FutureModel` subclass (`main.py` asserts this).

### Model args/configs

- Define model-specific args in static `get_parser(parser)`.
- Prefer updating the passed parser in-place (backward compatibility exists, but in-place updates are safer).
- Rehearsal models should use `utils.args.add_rehearsal_args(parser)`.
- Model configs load from `models/config/<model>.yaml`.
- `--model_config` accepts `default` and `best` (`base` is an alias of `default`).
- `--model_config best` for rehearsal models requires valid `--buffer_size`.
- A model config may set `dataset_config`; when present, it overrides CLI `--dataset_config` during config loading.

### Model author checklist

1. Add `models/<name>.py` (single primary model class).
2. Add `NAME` + accurate `COMPATIBILITY`.
3. Implement `observe` with stable return contract.
4. Add model parser args only when needed.
5. Add/adjust `models/config/<name>.yaml` if applicable.
6. Add targeted tests.

## 7) Datasets

### Discovery

- `datasets/__init__.py` supports both registered and legacy-discovered datasets.
- Name normalization also tolerates `_`/`-` differences.

### Required dataset contract

Use `ContinualDataset` (or `GCLDataset` for general-continual).

Required fields include:

- `SETTING`, `N_CLASSES_PER_TASK`, `N_TASKS`, `SIZE`, `N_CLASSES`, `AVAIL_SCHEDS`.

Required methods include:

- `get_data_loaders`, `get_backbone`, `get_transform`, `get_loss`,
- `get_normalization_transform`, `get_denormalization_transform`.

`GCLDataset` constraints (enforced via argument checks):

- `n_epochs == 1`,
- `eval_future == 0`,
- `enable_other_metrics == 0`,
- `noise_rate == 0`.

### Loader contract

For standard continual datasets, train samples must return:

- `(augmented, label, non_augmented)`

`store_masked_loaders(...)` is the expected splitter and handles:

- task masking,
- optional validation split,
- class permutation,
- noisy labels,
- partial labels (CSSL),
- biased-setting extras.

Extra-field behavior:

- Dataset extra return fields are propagated as kwargs to `model.observe` by name.
- Example: noisy-label flow adds `true_labels`; models that use it must include it in `observe` signature (or `**kwargs`).

`utils/training.py` asserts non-GCL loaders satisfy wrapper expectations.

### Dataset configs/defaults

- Dataset configs live in `datasets/configs/<dataset>/<config>.yaml`.
- `--dataset_config` selects config.
- Dataset code can set CLI defaults via `@set_default_from_args(...)`.

### Dataset author checklist

1. Add `datasets/<name>.py`.
2. Implement required fields/methods.
3. Preserve train tuple contract.
4. Prefer `store_masked_loaders` unless custom setting requires otherwise.
5. Add config file(s) and tests if needed.

## 8) Backbones

### Registration and API

- Register with `@register_backbone('<name>')`.
- Dynamic args are inferred from constructor/function signature.
- Prefer inheriting `MammothBackbone`.
- `forward(x, returnt=...)` should support at least:
  - `out`, `features`, `both`, `full`.
- Avoid argument-name collisions with existing parser args: dynamic parser building skips duplicate names.

### Backbone checklist

1. Implement under `backbone/`.
2. Register it.
3. Keep signature parser-friendly.
4. Validate dynamic args and model usage.

## 9) Argument/config precedence

Parser/config resolution is staged and non-trivial. In practice, treat priority as:

1. explicit CLI args,
2. model config values (`--model_config`, including dataset-specific `best` entries),
3. checkpoint-loaded defaults for unspecified args (`--loadcheck` path),
4. dataset config/default values and parser defaults,
5. code-level defaults.

If changing parser/config behavior, verify with `tests/test_model_config.py` and `tests/test_dynamic_params.py`.

When in doubt, inspect the actual flow in `main.py` (`parse_args`, `load_configs`, `update_cli_defaults`) because defaults are injected in multiple stages.

## 10) Validation playbooks

Model changes:

1. `uv run pytest tests/test_import.py tests/test_module_register.py tests/test_model_config.py`
2. relevant model test file(s)
3. `uv run pytest tests/test_basic_functionality.py`
4. if CSSL/partial labels are touched: `uv run pytest tests/test_cssl_support.py`

Dataset changes:

1. `uv run pytest tests/test_datasets.py tests/test_validation.py tests/test_noisy_label.py`
2. add `tests/test_bias.py` if biased path touched
3. run reload tests only if needed

Backbone/dynamic arg changes:

1. `uv run pytest tests/test_module_register.py tests/test_dynamic_params.py`

Parser/config changes:

1. `uv run pytest tests/test_model_config.py tests/test_dynamic_params.py tests/test_import.py`
2. `uv run python main.py --model sgd --dataset seq-cifar10 --help`

Docs-only changes:

1. docs build command from section 3.

## 11) Style and change discipline

- Match local file style.
- Avoid unrelated rewrites.
- Preserve public and hook signatures.
- Prefer `logging` over `print`.
- Keep side effects explicit and intentional.
- Update tests/docs alongside contract changes.

## 12) Known pitfalls

- Use `main.py` (not `utils/main.py`).
- Runtime model config path is `models/config/*.yaml`.
- Runtime dataset config path is `datasets/configs/<dataset>/*.yaml`.
- Optional dependencies may break import/discovery of specific models.
- Importing `models`/`datasets` package modules changes process cwd to repo root (`os.chdir(mammoth_path)` in package init).
- `--load_best_args` exists but is deprecated/untested; prefer `--model_config`.
- `--eval_future` without `FutureModel` fails by assertion.
- Non-GCL datasets are expected to behave like `store_masked_loaders` outputs.
- `backbone` typing alias still mentions `returnt='all'`; docs/models generally use `returnt='full'`. Keep backward compatibility if touching this interface.

Remote checkpoint/cache nuances:

- `--loadcheck` accepts local files and remote URLs (including Hugging Face resolve links).
- `TAK` Fisher loading accepts local paths, HTTP(S) base URLs, and `hf://...` sources via `--fisher_cache` when `--load_fisher=1`.
- If a remote URL returns a Git LFS pointer text file instead of artifact bytes, loading fails (use a direct raw/resolve artifact URL).

## 13) Workflow expectations for agents

- Read this file before editing.
- Keep changes minimal and task-focused.
- Avoid touching caches/artifacts (`data/`, `checkpoints/`, `docs/_build/`) unless task explicitly requires it.
- For artifact sharing, prefer `scripts/upload_to_hf.py` over ad-hoc upload snippets.
- For non-doc changes, run at least one focused test playbook plus a smoke run.
- Suggested smoke run: `uv run python main.py --model sgd --dataset seq-cifar10 --lr 1e-3 --n_epochs 1 --batch_size 2 --non_verbose 1 --num_workers 0 --debug_mode 1`
- Always report validation commands actually executed.

---
> Source: [aimagelab/mammoth](https://github.com/aimagelab/mammoth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
