## cherimoya

> Compact sequence-to-function deep learning model (DNA → genomic profile / counts)

# Cherimoya

Compact sequence-to-function deep learning model (DNA → genomic profile / counts)
with custom Triton kernels and an end-to-end CLI.

## Layout
- `cherimoya/`       — model + kernels. Public API is **only** `Cherimoya`,
                       `CheriBlock`, `EMA` (see `__init__.py`); everything else
                       is private and may break between versions.
- `cherimoya_cli/`   — `cherimoya` console script. Subcommands in
                       `commands/`; **all CLI defaults live in
                       `cherimoya_cli/defaults.py`** — edit there, not in
                       individual subcommand files.
- `tests/`           — pytest suite. CPU-only by default; CUDA/Triton tests
                       are auto-skipped when a GPU isn't visible.
- `docs/`            — Sphinx (furo). Built on Read the Docs.

Ignore `.ipynb_checkpoints/` directories anywhere in the tree — stale
Jupyter autosaves, gitignored, not authoritative. Never read or edit them.

## Commands
```bash
pip install -e .                       # dev install
pip install -e .[docs]                 # add Sphinx + furo + copybutton

pytest                                 # full suite; CUDA/triton auto-skip
pytest -m "not cuda and not triton"    # force CPU-only
pytest tests/test_cheri.py::test_name -x -q   # one test, fail-fast, quiet

cd docs && sphinx-build -W -b html . _build/html   # strict docs build

cherimoya --help                       # CLI entry (see docs/cli.rst)
```

Subcommands: `pipeline-json`, `pipeline`, `negatives`, `fit`, `evaluate`,
`attribute`, `seqlets`, `marginalize`, `batch`.

## Conventions
- **Indentation: tabs**, not spaces. Don't let formatters convert.
- **Commit messages: UPPERCASE verb prefix** — `ADD`, `FIX`, `UPDATE`,
  `REWRITE`, `RELEASE`, `REMOVE`. Match the existing `git log` style.
- **Docs/README accuracy is a hard rule**: every concrete detail
  (defaults, versions, flags, behaviors) must be confirmable from source.
  If you can't verify it, omit it — don't assume.

## Gotchas (non-obvious)
- **`torch.compile` is disabled in tests** via `TORCH_COMPILE_DISABLE=1` in
  `tests/conftest.py`. Don't re-enable it for unit tests — it adds minutes
  of warm-up and isn't what those tests verify.
- **Three forward paths must stay numerically equivalent (~1e-5 max-abs):**
  CPU PyTorch, Triton training fwd+bwd, Triton inference megakernel.
  Touching any of them requires running `test_cheri.py` cross-path checks.
  The megakernel dispatches only when `torch.is_grad_enabled() is False`
  and `expansion * n_filters` is a multiple of 16.
- **Checkpoint format = config + state_dict bundle**, not a pickled module.
  Use `Cherimoya.load(...)` / `model.save(...)`. Pre-0.1.0
  `torch.save(model, ...)` checkpoints are not loadable and won't be
  supported.
- **Saved weights are the EMA shadow**, not the live training params.
  `Cherimoya.load` returns the EMA snapshot — compare against the EMA
  validation number in the training log, not the mid-epoch loss.
- **Reproducibility contract:** peak/negative sampling is a pure function
  of `(seed, epoch, index)`. `num_workers > 1` must produce the same batch
  sequence as `num_workers = 1`. Don't introduce worker-local RNG state.
- **`cherimoya pipeline` pre-flight check:** local input paths are
  validated before any expensive work starts; remote paths (`http://`,
  `https://`, `s3://`, `gs://`) are skipped. If a JSON key points at a
  file the pipeline itself is supposed to produce later, set it to
  `null` so the pre-flight doesn't reject the run.

## Dependencies worth knowing
- `tangermeme` — `io.extract_loci`, `predict`, `saturation_mutagenesis`,
  `variant_effect`, `ersatz`, `seqlet.recursive_seqlets`,
  `match.extract_matching_loci`. Don't reimplement.
- `bpnet-lite` — `Logger`, `MNLLLoss`, `log1pMSELoss`, `ControlWrapper`,
  `CountWrapper`, `ProfileWrapper`, `marginalization_report`.
- `bam2bw` — streams BAM/SAM/fragment → bigWig; supports remote inputs
  (S3/HTTPS/GCS via range requests).
- `Muon` optimizer comes from `torch.optim` (PyTorch ≥2.9), not a
  third-party package.

Nothing runs as a git hook — run `pytest` and the strict Sphinx build
(both above) before opening a PR.

---
> Source: [jmschrei/cherimoya](https://github.com/jmschrei/cherimoya) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
