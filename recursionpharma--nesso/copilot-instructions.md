## nesso

> Guidance for coding agents working in the **nesso** repo. Nesso-1 is a fast,

# AGENTS.md

Guidance for coding agents working in the **nesso** repo. Nesso-1 is a fast,
structure-based protein–ligand binding-affinity model. Inference takes a protein
sequence + a ligand (SMILES / CCD code / SDF) and predicts an affinity scalar.

For human-facing usage, start with `README.md` and `docs/prediction.md`. This file
captures the operational details and gotchas that aren't obvious from reading the source.

## Environment

- **Python `>=3.10,<3.13` is a hard constraint** (see `pyproject.toml`). The package
  will not install on 3.13+. This is the most common silent setup failure — check the
  interpreter version first when an install or import misbehaves.
- Install for development from a clone:
  ```bash
  uv sync                 # or: pip install -e .
  ```
- GPU: nesso relies on PyTorch. For GPU runs, install a CUDA-enabled PyTorch build per
  the official PyTorch instructions before installing nesso. The optional
  `nesso[kernels]` extra enables NVIDIA cuEquivariance kernels (CUDA-12 only); if it
  fails to install or you're on CPU, use the default install and/or pass `--no_kernels`.

## Running inference

The package installs a single `nesso` CLI (a `click` group; the only command is
`predict`). The entry point is `nesso.main:cli`.

```bash
nesso predict <input.yaml | dir/> --out_dir <out>      # single file or a directory of YAMLs
nesso predict <...> --accelerator gpu --devices 1       # GPU
nesso predict <...> --save_metadata                     # also dump pairwise tensors
```

Common flags (full list in `docs/prediction.md`): `--checkpoint` (local directory with
`hparams.json` + `model.safetensors`, bypasses the cached download), `--ccd` (custom CCD
pickle), `--cache` / `$NESSO_CACHE` (asset cache location), `--no_kernels`, `--num_workers`,
`--recycling_steps`.

Input YAML schema and the three ligand input modes (SMILES / CCD / SDF) are documented in
`docs/prediction.md`; runnable examples live in `tutorial/`.

## Output contract

Predictions are written to:

```
{out_dir}/predictions/{record_id}/affinity.json          # always
{out_dir}/predictions/{record_id}/predictions.safetensors # only with --save_metadata
```

- **`record_id`** is the input YAML's filename stem. When batching many pairs, name your
  YAMLs deterministically — `record_id` is the join key for collecting results back.
- **`affinity_pred_value` = log10(IC50 / µM)** — *lower means stronger* binding
  (≈ −3 → 1 nM, 0 → 1 µM, +2 → 100 µM). Convert to pIC50/pchembl with `6 - affinity_pred_value`.
- `affinity_pred_value1` / `affinity_pred_value2` are the two ensemble members; their
  spread `|value1 − value2|` is a cheap uncertainty proxy (small = confident).
- `affinity_probability_binary` is a separate binder/non-binder classifier head — usable
  as a confidence signal independent of the regressed potency.
- `entropy_{pp,pl,ll}` are distogram entropies for protein–protein / protein–ligand /
  ligand–ligand pairs. `predictions.safetensors` (with `--save_metadata`) holds the
  Pairformer `z` representation and distogram tensors; see `tutorial/extract_features.py`.

## Gotchas

- **Checkpoints load from `hparams.json` + `model.safetensors`.** Default weights are
  pulled from the Hugging Face model repo (`recursionpharma/nesso`) and instantiated via
  `Nesso1.from_pretrained()`. Use `--checkpoint` to point at a local directory containing
  both files instead of the cached Hub download.
- **Model weights are downloaded/cached on first run.** If auto-download is unavailable in
  your environment, point `--checkpoint` (and `--ccd` if needed) at local copies; nesso
  then skips the corresponding downloads (the public ESM-2 weights are still fetched/cached).
- Set `NESSO_CACHE` to keep large cached assets off the default `./.cache/`.

## Code style & checks

### Tests

```bash
pytest                 # run the suite (config: [tool.pytest.ini_options] in pyproject.toml)
```

The `tests/` suite is **CPU-only and weight-free** — it covers the input/output
plumbing around the model (YAML parsing & validation, `z` feature slicing, padding,
pocket cropping, safetensor I/O), not the model forward pass (which needs the gated
checkpoint + a GPU and is intentionally out of scope for CI).

A few tests build a *protein* chain, which requires standard-residue mols from the CCD
pickle. They **skip automatically** unless a CCD pickle is found via `NESSO_CCD`,
`$NESSO_CACHE/ccd.pkl`, or under the Hugging Face cache in
`$NESSO_CACHE/huggingface/...` (defaulting to `./.cache/huggingface/...`) — so a bare
`pytest` run is green (some skipped), and pointing `NESSO_CCD` at a pickle exercises
the full front-end.

`ccd.pkl` is downloaded into the Hugging Face cache under `.cache/huggingface/`
automatically on your first `nesso predict` run, so after running the model once a
plain `pytest` picks it up; otherwise point `NESSO_CCD` at an existing copy.

### Lint / format

- Linting/formatting is **ruff**, wired through `pre-commit` (`.pre-commit-config.yaml`);
  `ruff-check --fix` and `ruff-format` run on commit. Run `pre-commit run --all-files`
  before proposing changes.

## Package layout

```
nesso/main.py          # CLI (click): `predict`
nesso/data/            # I/O + featurization: yaml_input, io, esm, inference, crop, pad, writer, types, const
nesso/model/           # model definition + weights loading
tutorial/               # runnable YAML examples + extract_features.py
tests/                  # CPU-only, weight-free tests for the I/O plumbing
docs/prediction.md      # CLI flags, YAML schema, output fields (authoritative)
```

---
> Source: [recursionpharma/nesso](https://github.com/recursionpharma/nesso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
