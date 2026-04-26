## mil

> >-


# Histopathology MIL project context

## What this repo is

- **Task**: Multiple Instance Learning (MIL) on **pre-extracted patch features** for **PANDA prostate WSIs**, predicting **ISUP grade groups** (6 classes: 0–5).
- **Not in-repo**: Full WSI I/O or stain normalization; upstream **TRIDENT** (or similar) produces **per-slide `.h5` bags** (`features`, optional `coords`).

## Stack and layout

- **Python / PyTorch**: training in `mil/train.py`; models in `mil/model.py` (attention / gated attention MIL, masking for variable bag sizes).
- **Data**: `mil/data.py` — `H5BagDataset` / caching, collation, optional patch subsampling and attention-biased sampling; labels via `pandas` tables aligned to slide IDs.
- **Loss**: `mil/loss.py` — includes ordinal-aware objectives where used.
- **Metrics / CV**: `mil/utils.py`; summarized results may live under `figures/` (e.g. CSVs).
- **Visualization**: `mil/attention_map.py`, `mil/vit_attention_viz.py` for attention / explainability — preserve coordinate semantics when mapping back to slides.

## Conventions for assistants

- Match existing style: `from __future__ import annotations`, type hints, `pathlib.Path` for filesystem paths.
- Treat each slide as a **bag** of instances; respect **padding masks** so attention and pooling ignore invalid positions.
- When suggesting **cluster paths or PBS/TRIDENT** workflows, follow **`.cursor/rules/trident-paths.mdc`** for canonical MetaCentrum locations.
- Prefer **small, task-focused changes**; do not refactor unrelated modules or add unsolicited markdown docs.
- For **medical / diagnostic** claims: distinguish **code behavior** from **clinical interpretation**; do not overstate validation from local CV alone.

## Useful mental model

**WSI → patches → frozen encoder features (.h5) → MIL aggregator → ISUP class (ordinal or CE as configured).** Debugging usually traces: missing or mismatched `.h5`, label CSV alignment, `feat_dim` vs encoder, mask/batch shapes, and train vs eval patch caps.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/filip-sec) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
