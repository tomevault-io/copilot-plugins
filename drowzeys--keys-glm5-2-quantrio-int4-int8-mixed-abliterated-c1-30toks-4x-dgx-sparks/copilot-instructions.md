## keys-glm5-2-quantrio-int4-int8-mixed-abliterated-c1-30toks-4x-dg

> git clone https://github.com/drowzeys/keys-GLM5.2-Quantrio-INT4-INT8-Mixed-Abliterated-C1-30toks-4x-DGX-Sparks

# Agent instructions — standing GLM-5.2 Quantrio ablit

## Single path (do this)

```bash
git clone https://github.com/drowzeys/keys-GLM5.2-Quantrio-INT4-INT8-Mixed-Abliterated-C1-30toks-4x-DGX-Sparks
cd keys-GLM5.2-Quantrio-INT4-INT8-Mixed-Abliterated-C1-30toks-4x-DGX-Sparks

# REQUIRED — kernels + public image + HF ablit weights + hub symlink + verify
bash scripts/oneshot.sh

# Multi-node TP=4 (edit WORKERS):
WORKERS="10.100.10.1 10.100.10.2 10.100.10.3" bash scripts/oneshot.sh --fanout

# Launch SPEED=1 (C1≈30) and optional 32/32 suite:
bash scripts/oneshot.sh --launch
# or everything:
WORKERS="..." bash scripts/oneshot.sh --all
```

## Hard rules

1. **Do not** download QuantTrio stock as the serve tree.
2. **Do not** invent hub names — launcher needs `hub/glm52-int4-int8mix` → abliterated.
3. **Do** get `verify_ablit_weights.py` **PASS** on **every** rank.
4. **Do** force `enable_thinking: false` for refusal probes / published bypass claim.
5. Ablit is **offline in weights** — not applied at launch. Missing bypass = wrong tree / thinking / mixed ranks.

## Layout

| Path | Purpose |
|---|---|
| `scripts/oneshot.sh` | Master one-shot |
| `scripts/oneshot_install_weights.sh` | HF ablit download + verify + hub |
| `scripts/install_kernels.sh` | `kernels/` → `~/glm-triton` |
| `scripts/pull_image.sh` | Public GHCR image |
| `serve/launch-keyspark.sh` | SPEED=1 TP=4 serve |
| `scripts/serve-capture.sh` | Capture serve (rebuild direction) |
| `scripts/project_residual.py` | Path B re-project o_proj L65–77 |
| `scripts/hybrid_overlay.py` | Early-stock hybrid helper |
| `recipe/` | DIRTY_SHARDS, ABLIT_META, direction.pt |
| `kernels/` | Sparse-MLA bind-mounts |
| `patches/` | Capture + indexer fix |
| `image/` | Bake mods (optional rebuild) |
| `docs/` | ONE_SHOT, RECIPE, SPEED1, PARTIAL_BYPASS, … |

## Docs priority

1. `README.md` / `docs/ONE_SHOT.md` / `docs/IMPORTANT.md`
2. `docs/PARTIAL_BYPASS.md` if &lt;100% bypass
3. `docs/RECIPE.md` for SRA / Path B only

---
> Source: [drowzeys/keys-GLM5.2-Quantrio-INT4-INT8-Mixed-Abliterated-C1-30toks-4x-DGX-Sparks](https://github.com/drowzeys/keys-GLM5.2-Quantrio-INT4-INT8-Mixed-Abliterated-C1-30toks-4x-DGX-Sparks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
