## rijksmuseum-se

> Exploratory project for visualizing the Rijksmuseum collection's 831K artwork embeddings

# Rijksmuseum Semantic Explorer

Exploratory project for visualizing the Rijksmuseum collection's 831K artwork embeddings
using MLX-accelerated dimensionality reduction on Apple Silicon.

## Project Structure

```
lib/embeddings.py    — Shared: load, decode, normalize, sample embeddings + metadata
notebooks/           — Jupyter notebooks with Plotly interactive visualizations
patches/             — Local patches for vendored dependencies (applied after uv sync)
data/                — Symlinks to ../rijksmuseum-mcp-plus/data/{embeddings,vocabulary}.db
output/              — Generated HTML, coords (.gitignore'd)
```

## Data

- **embeddings.db**: 831,667 artwork embeddings (multilingual-e5-small, 384 dims, int8 quantized)
- **vocabulary.db**: Structured metadata (titles, creators, types, subjects, materials, techniques)
- Both are symlinked from `../rijksmuseum-mcp-plus/data/`

## Key Patterns

- Embeddings stored as int8 BLOBs in SQLite → decode with `struct.unpack` → float32 → L2-normalize
- Vocabulary DB uses integer-encoded schema (field_lookup + mappings + vocabulary tables)
- Batch queries in chunks of ~990 to stay within `SQLITE_LIMIT_VARIABLE_NUMBER`

## Commands

```bash
uv sync && bash patches/apply-patches.sh   # Install dependencies + apply local fixes
uv run jupyter lab                          # Launch notebooks
```

## Patches

Local fixes to vendored dependencies live in `patches/`. Run `bash patches/apply-patches.sh`
after every `uv sync` to re-apply them. The script is idempotent (skips already-applied patches).

- **mlx-vis-pacmap-memory-fix.patch** — Fixes O(n) Metal memory accumulation in `_brute_knn` that
  caused 20+ GB RAM / 7 GB swap at 200K points. Switches to per-chunk numpy copy (umap pattern)
  and removes redundant numpy↔MLX round-trips in `fit_transform`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kintopp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
