## cs-senior-project

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Floor Plan Parser Engine — the goal is to turn a hand-drawn 2D floor plan into RoomPlan-style JSON for Three.js 3D visualization. A pretrained GAT-Net (Graph Attention Network) predicts room dimensions from the spatial graph of a floor plan. Currently the pipeline reads from the ResPlan dataset (ground-truth polygons) as a working POC; the image-parsing front-end (hand-drawn sketch → room polygons) is a planned future milestone. Python 3.11, macOS (MPS) or CPU.

## Commands

```bash
# Setup
python3.11 -m venv venv && source venv/bin/activate && pip install -r requirements.txt
# Place ResPlan.pkl (~278 MB, from Kaggle) in project root

# Generate floor plan JSON
python parse_plan.py --layout-id 0                    # → output/layouts/layout_{id}.json
python parse_plan.py --layout-id 42 --output custom.json

# Verify predictions against ground truth
python verify_layout.py --layout-id 0                 # table + plot → output/plots/
python verify_layout.py --layout-id 0 --no-plot       # table only
```

No test suite exists. Verification is done via `verify_layout.py` (IoU scores + visual comparison).

## Architecture

**Four-stage pipeline in `parse_plan.py`:**

1. **Preprocess** — extract room polygons from ResPlan entry, snap walls (tolerance=1.0), morphological close for watertight geometry, normalize room type labels
2. **Graph Construction** — build room graph (NetworkX → PyG, 9 features: 7 one-hot room type + 2 z-score centroids) and boundary graph (3 features: node type + 2 z-score centroids)
3. **GAT-Net Inference** — 4 GAT layers on room graph + 2 on boundary graph, max-pool boundary features and broadcast to rooms, predict width/height per room
4. **JSON Export** — clip interior rooms to boundary polygon (skip balconies), serialize as RoomPlan-style JSON with walls, doors, metadata

**Key files:**

| File | Role |
|------|------|
| `parse_plan.py` | Main CLI, all 4 pipeline stages, `GATNetCompat` class |
| `verify_layout.py` | IoU comparison and matplotlib visualization |
| `Floor_Plan_Generation_using_GNNs/model.py` | `GATNet` architecture (DO NOT modify) |
| `Floor_Plan_Generation_using_GNNs/utils.py` | `FloorPlan_multipolygon`, `Handling_dubplicated_nodes`, `centroids_to_graph` (DO NOT modify) |

## Critical Technical Details

**PyG version mismatch:** The checkpoint uses older PyG with separate `lin_src`/`lin_dst` in GATConv. Current PyG 2.7 uses single `lin`. Fix: pass `in_channels` as tuple `(n, n)` to create bipartite-style layers. See `GATNetCompat` in `parse_plan.py`.

**Normalization constants (MUST match training distribution):**
```
Room graph:      mean_x=127.53, std_x=71.65, mean_y=131.35, std_y=55.24
Boundary graph:  mean_x=128.33, std_x=90.13, mean_y=128.10, std_y=73.62
```
Changing these to per-graph stats degrades predictions. Missing boundary normalization causes predictions ~1800 instead of ~30-180.

**Room type embeddings (7-class):** living=0, bedroom=1, kitchen=2, bathroom=3, balcony=4, storage=5, entrance=6. `front_door` goes in the boundary graph, not the room graph.

**Coordinate system:** pixel-based 0-256, y-axis inverted. The `scale()` function in utils.py handles this.

**Concatenation residuals** (not skip connections) in GAT layers prevent over-smoothing — each layer concatenates its output with the original input.

## Conventions

- Room IDs: `{type}_{index}` (e.g., `bedroom_0`)
- Polygons serialized without closing duplicate point
- Wall depth constant: 0.15m (for Three.js 3D extrusion)
- Interior rooms clipped to boundary with 3px inset; balconies are exempt
- Device priority: MPS (Apple Silicon) → CPU
- Generated outputs go in `output/layouts/` and `output/plots/` (gitignored)
- `Floor_Plan_Generation_using_GNNs/` contains upstream model code — do not modify `model.py` or `utils.py`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/supachaidev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/supachaidev)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
