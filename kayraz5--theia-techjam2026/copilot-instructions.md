## theia-techjam2026

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Handoff note: the codebase is complete and its author has stepped back. The standing instruction is to
**explain and analyze, not modify** — treat code edits as out of scope unless the user explicitly changes
that. The original phase-1 brief is preserved at `docs/PROJECT_BRIEF.md`; the full build spec it grew into
is reflected in `README.md`. Results live in `results/RESULTS.md`.

## What this is

A robust AI-generated-image detector (hackathon deliverable): image → `p(AI-generated) ∈ [0,1]`, evaluated
as a ROC-AUC grid over 6 transform families × severities (JPEG/blur/resize/noise/color/crop). It reproduces
the 5th-place NTIRE 2026 recipe: squish-resize 384², SigLIP2-giant vision tower (1.164 B params, asserted
< 2 B at every startup), mean-pool over final-layer patch tokens (no CLS, no attention pooling), single
linear head. Trained with `distortion_prob = 1.0` (every training image gets 1–3 random distortions).

## Commands

Everything runs inside the uv venv: `source .venv/bin/activate` (Python 3.12; recreate with
`uv venv --python 3.12 .venv && uv pip install -r requirements.txt`).

```bash
pytest tests/test_degradation.py         # unit tests for the transform pipeline (fast)
pytest tests/test_pipeline_smoke.py      # end-to-end train→eval→predict with a stub backbone (~2 min, CPU)
pytest tests/test_degradation.py::test_jpeg_quality_monotonic   # single test

scripts/prepare_data.sh                  # one-time ~5 GB data pull (seeded slices; see Data below)
python -m src.data.build_val --config configs/frozen_siglip2_giant.yaml   # val manifest + exclusion hashes
python -m src.train    --config configs/frozen_siglip2_giant.yaml         # frozen arm (asserts no leakage first)
python -m src.evaluate --config configs/frozen_siglip2_giant.yaml         # 15-condition AUC grid + plots
python -m src.compare  results/<arm1> results/<arm2> ...                  # cross-arm comparison table
python scripts/make_results.py                                            # regenerate results/RESULTS.md
python predict.py --image_dir <dir> --output preds.json                   # the deliverable CLI
```

Configs are the four experiment arms; `mode: frozen|lora` switches training style. Long runs: launch with
`nohup ... > logs/<name>.log 2>&1 &` and read progress with `tr '\r' '\n' < logs/<name>.log | grep '\[features\]'`
(progress bars use `\r`). **The Mac must stay awake** (`caffeinate -i <cmd>`) — sleep pauses MPS jobs silently.

## Architecture (the one path everything shares)

```
config YAML
  └─ src/common.py            build_backbone() (param assert + fallback), squish_resize, metrics
  └─ src/data/registry.py     every source → list[Sample(path,label,source)]; build_train() assembles the mix
        exclusion.py          pixel-content hashes (64×64 RGB + orig size); ExclusionList.assert_disjoint()
                              is called by src/train.py at EVERY run before any feature is extracted
  └─ src/degradation/         transforms.py = the 6-family table, shared verbatim by training augmentation
                              AND eval conditions (EVAL_CONDITIONS); official.py wraps the vendored NTIRE
                              pipeline (third_party/aug_utils_train) — optional backend, not default
  └─ src/features.py          frozen-feature extraction with .npz caching in data/features/
                              cache key = (backbone, image_size, tag, exact sample-path list)
  └─ src/train.py             frozen: train linear head on cached features (seconds once extracted);
                              lora: end-to-end rank-32 LoRA. Checkpoints on best ROBUST val AUC
  └─ src/evaluate.py          per-condition grid on a seeded 2,000-image subset + 1,000 alt-reals,
                              full-set clean AUC (incl. deduplicated), heatmap, thresholds/ROC,
                              FP/FN contact sheets → results/<arm>/eval/
```

The backbone wrapper (`src/models/backbone.py`) is family-generic (SigLIP2 / DINOv3 / CLIP) — patch-token
slicing per family is the only model-specific code, so all arms share train/eval unchanged.

## Data layout (not in git; ~15 GB under data/)

- `data/sid_set/` — 8 of 249 SID_Set parquet shards + materialized PNGs (4,000 train imgs; label 2 "tampered" excluded)
- `data/wildfake/` — csv labels + images pulled from ModelScope zips via `scripts/remote_zip.py`
  (HTTP-Range extraction; the full dataset is 1.29 TB — never download whole archives)
- Held-out validation = organisers' designated subset: 4,998 COCO val2017 reals + 8,843 DALL-E Advanced
  (= `dalle3.csv`) fakes. **Never trained on**; enforced by hash + structural refusals in registry.py
- Alt-real set: 1,000 WildFake ImageNet reals — the COCO-shortcut control, reported next to every number
- `data/features/*.npz` — feature caches; deleting them just costs re-extraction (~5.4–12 img/s on this M5 Pro)

## Facts future sessions will need (hard-won, non-obvious)

1. **Results state (2026-08-25).** Frozen SigLIP2-giant on the spec'd train mix: clean AUC 0.931 full-set,
   worst cell resize@0.25 = 0.810. Data ablation (head retrained on SID_Set only, same features): clean
   0.992, worst 0.989, max delta 0.004 — meets all targets. Cause: the WildFake train slice has a format
   confound (DALL-E 2 fakes are all 512² PNG, LAION reals are JPEG → head learns "JPEG → real"; that slice
   alone scores AUC 0.42 on validation). Both grids are committed: `eval/auc_grid.csv` and
   `eval/auc_grid_sid_only_ablation.csv`. **The headline-arm choice (mixed vs SID-only) is an open user decision.**
2. **The designated validation set is internally duplicated**: only 3,719 of the 8,843 DALL-E files are
   unique images. `summary.json` carries both `clean_auc_full_set` and `clean_auc_full_set_dedup`.
3. **Dataloader workers**: > 2 spawned workers on this 64 GB unified-memory Mac starves the GPU
   (6 workers → ~1 img/s; 2 → ~6–12 img/s). Defaults are 2 everywhere; only raise on discrete-GPU boxes.
4. **Never report accuracy alone** — class ratio is ~1:1.77 (majority baseline ≈ 0.65). AUC is headline;
   accuracy only with balanced accuracy + baseline beside it. Always report COCO-real AND alt-real AUC.
5. Feature-cache tags: eval subset now uses `valsub_`/`altsub_` prefixes (a historical collision with the
   full-set `val_clean` tag caused a redundant 24-min re-extract; fixed, but old caches keep old names).
6. `google/siglip2-giant-opt-patch16-384` vision tower = 1.164 B (fits); DINOv3 arm is blocked on gated-repo
   access (needs HF_TOKEN + approval). CLIP baseline runs at 224 px. CIFAKE stays off (32² px, smoke-test only).
7. macOS quirks: no `timeout` command; multiprocessing is spawn-based (worker code must be importable, no
   heredoc `__main__`); `sed -i ''`.

## Still not done (spec steps 6–7 leftovers)

- LoRA arm (`configs/lora_siglip2_giant.yaml`) — only GPU-hungry item left (~2–3 h on this machine)
- CLIP ViT-L/14 zero-shot linear-probe baseline arm + `src.compare` table across all arms
- DINOv3 comparison (blocked on gated access)
- `notebooks/` is empty; figures live in `results/<arm>/eval/`

---
> Source: [kayraz5/THEIA-techjam2026](https://github.com/kayraz5/THEIA-techjam2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
