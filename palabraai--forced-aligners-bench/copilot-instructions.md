## forced-aligners-bench

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A benchmark that ranks forced aligners (Qwen3-ForcedAligner, Seamless UnitY2, WhisperX) **without ground-truth timestamps**. Methodology: run each aligner, crop audio at its word boundaries, re-transcribe every crop with Whisper large-v3-turbo, compute WER. Tighter alignment ⇒ lower WER. See `README.md` for the full rationale and `RESULTS.md` for the last published numbers.

## Commands

```bash
# Install (monorepo — the top-level pyproject.toml installs everything)
pip install -e .

# Download FLEURS test split for EN/ES/FR/RU/KO
bash get_data.sh

# Extract alignments with ONE aligner on ONE language (multi-GPU capable)
python extract_alignment.py cache/data/fleurs/en/test/meta.csv \
    -a qwen3 -l en -g 0 -g 1 -p 3        # -g is repeatable, -p = procs/GPU

# All aligners × all downloaded languages
bash run_aligners.sh

# Benchmark (requires meta.{aligner}.csv files to already exist in test_dir)
python bench_alignment.py cache/data/fleurs/en/test/ \
    -s "1,-1" -l en --batch-size 64 --filter-perfect --strict

# Benchmark all downloaded languages with the headline config
bash run_transcribe.sh
```

There is **no test suite, no linter, and no CI** at the repo root. Sub-package `pyproject.toml` files under `aligners/*/` are provenance from upstream (Poetry) — they are *not* installed; the root `pyproject.toml` (hatchling) installs `aligners/` and `dataprep/` as the two packages.

## Architecture

The pipeline is two-stage, split across two top-level scripts that communicate only via CSV files on disk:

1. **`extract_alignment.py`** reads `meta.csv` (columns: `path_abs`, `transcription`), fans out rows over GPUs via `multiprocessing.Process` (each worker sets `CUDA_VISIBLE_DEVICES` itself before importing torch models), runs one aligner, and writes `meta.{aligner}.csv` beside the input — same rows + an `alignment` column holding a JSON list of `{word, start, end}`. Workers write `.part_{gpu}_{pid}.csv` shards that `merge_parts()` stitches back into the original row order.

2. **`bench_alignment.py`** discovers *all* `meta.{aligner}.csv` files in a test dir by filename convention (`ALIGNER_TYPES = ["qwen3", "whisperx", "seamless"]`), builds a flat list of `CropTask`s across aligners × slice specs × rows, loads a single Whisper model, and transcribes every crop batched. `--filter-perfect --strict` keeps only utterances where *every* aligner reconstructed the ground-truth transcript exactly (intersection on `path_abs`) — this is the fair-comparison mode that produces the headline numbers. Without `--strict`, each aligner is filtered independently (easier, incomparable across aligners).

### Aligner interface (the extension point)

All three aligner classes expose the same shape, and `extract_alignment.py` depends on it:

```python
aligner(audio, transcript, language) -> (pd.DataFrame[word, start, end, ...], raw_result)
```

- `ALIGNER_REGISTRY` in `extract_alignment.py` maps CLI keys (`qwen3`, `whisperx`, `seamless`) to classes. Adding a new aligner means: (a) new class with `__init__(device=...)` and `__call__(audio, transcript, language)`, (b) one line in the registry, (c) append to `ALIGNER_TYPES` in `bench_alignment.py` so it's discovered.
- Aligners are imported via `sys.path.append` of `aligners/<name>/` in `extract_alignment.py` — the three sub-packages are **not** on the normal Python path; they're siblings of the top-level package and the script inserts their directories. If you move files, update the `sys.path.append` lines at the top of `extract_alignment.py`.
- Each aligner normalises mixed audio inputs (path / dict / ndarray / tensor) to 1-D float32 @ 16 kHz in its own `prepare_audio()`. Keep that behaviour when editing — `align_single()` hands paths in directly.

### Vendored sub-packages (`aligners/*/`)

- **`aligners/qwen3/qwen3_asr/`** — trimmed copy of QwenLM/Qwen3-ASR (Apache-2.0). The actual Qwen3 model package is one level deeper at `aligners/qwen3/qwen3_asr/qwen_asr/`; `aligner.py` does `sys.path.insert(0, ...)` to reach it, then `from qwen_asr import Qwen3ForcedAligner`. Model weights fetch at runtime from HF (`Qwen/Qwen3-ForcedAligner-0.6B`).
- **`aligners/whisperx/patched_whisperx/`** — derivative of `m-bain/whisperx@b2d50a02` (BSD-2-Clause). Monkey-patches `transformers.utils.import_utils.check_torch_load_is_safe` at import time (CVE-2025-32434 guard); also `import_without_prints` silently strips `print()` calls from whisperx internals. Don't remove those patches without checking why they exist. Upstream whisperx is pulled from GitHub via the sub-pyproject — but that sub-pyproject is **not** the install surface; the actual whisperx dep lives under `wtpsplit` in the root `pyproject.toml`.
- **`aligners/seamless/seamless_unity_aligner/`** — custom trim of `facebookresearch/seamless_communication` + `facebookresearch/fairseq2@v0.2.x`, vendored under `deps/` specifically to (a) avoid building `fairseq2n`'s C++ native layer and (b) exclude non-commercially-licensed Seamless checkpoints (SeamlessExpressive, full unified Seamless, etc.). Only the MIT-licensed subset reachable from `AlignmentExtractor` is present. The aligner entrypoint (`__init__.py`) monkey-patches `AlignmentExtractor.extract_alignment` with a custom version that also returns per-char log-prob sums → used to emit a Viterbi-probability `score` column per word. See `THIRD_PARTY_LICENSES.md` before touching anything under `deps/`.

### Data layout

`dataprep/load_fleurs.py` downloads FLEURS from HuggingFace and reorganises into:

```
cache/data/fleurs/<whisper_lang_code>/<partition>/
    meta.csv                  # ground truth (path_abs, transcription, ...)
    wavs/*.wav
    meta.qwen3.csv            # ← written by extract_alignment.py (one per aligner)
    meta.whisperx.csv
    meta.seamless.csv
    bench_results.strict.csv  # ← written by bench_alignment.py
```

The script uses Whisper language codes as the external interface (`en`, `ru`, …) and maps to FLEURS codes (`en_us`, `ru_ru`, …) internally via `dataprep/utils/lang_remap.json`.

## Conventions worth knowing

- **Slice syntax for `--slice/-s`** is Python-style `"start,end"` where `None` is literal. `"1,-1"` drops first and last word (headline config); `"0,None"` keeps everything. Repeatable (`-s "1,-1" -s "2,-2"`) — all slices run against the same Whisper model load.
- **`--strict` requires `--filter-perfect`** (CLI enforces). Strict mode is the only apples-to-apples comparison because each aligner is evaluated on *its own* reconstructible utterances otherwise.
- **`run_transcribe.sh`** runs `"1,-1" --filter-perfect --strict` for every language directory under `cache/data/fleurs/`. The commented-out block at the top is the unfiltered variant — keep both if you edit.
- When editing Whisper inference in `bench_alignment.py`, note that `os.environ["CUDA_VISIBLE_DEVICES"]` is set inside `run_benchmark()` *before* model load — respect that ordering, torch won't pick up the change after CUDA has initialised.
- CSV `alignment` column is a JSON string. Use `parse_alignment()` from `bench_alignment.py` — it handles the `None`/`NaN`/string/list cases that arise from `pd.read_csv` round-tripping.

## When suggesting changes

- **New aligner**: add class → register in two places (see "Aligner interface" above) → verify `meta.{new_aligner}.csv` appears after a run → `bench_alignment.py` picks it up automatically with no further changes.
- **New language**: add to the `get_data.sh` language list, run it, then `bash run_aligners.sh` + `bash run_transcribe.sh`. Qwen3 requires an entry in `QWEN3_LANG_MAP` (`aligners/qwen3/qwen3_asr/aligner.py`) — unlisted languages raise; WhisperX needs a wav2vec2 model for that language to exist upstream.
- **Don't restructure `aligners/seamless/seamless_unity_aligner/deps/`** without re-reading `THIRD_PARTY_LICENSES.md` — the trim is deliberately scoped to the MIT-licensed surface.

---
> Source: [PalabraAI/forced-aligners-bench](https://github.com/PalabraAI/forced-aligners-bench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
