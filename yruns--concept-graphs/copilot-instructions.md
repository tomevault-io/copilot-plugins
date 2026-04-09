## concept-graphs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ConceptGraphs is a research project for building open-vocabulary 3D scene graphs from RGB-D sequences. It uses Grounded-SAM for 2D detection/segmentation, CLIP for visual features, and LLM/VLM for scene understanding and querying.

## Environment Setup

```bash
# Create conda environment
conda create -n conceptgraph anaconda python=3.10
conda activate conceptgraph

# Install package in editable mode
pip install -e .

# Configure and load environment variables
source env_vars.bash
```

### Environment Variables

Configure in `env_vars.bash` (auto-generated from template):

| Variable | Description | Default |
|----------|-------------|---------|
| `REPLICA_ROOT` | Replica dataset root path | `/Users/bytedance/Replica` |
| `CG_FOLDER` | Project root (auto-detected) | - |
| `GSA_PATH` | Grounded-Segment-Anything path | `${CG_FOLDER}/Grounded-Segment-Anything` |
| `LLM_MODEL` | LLM for query parsing | `gpt-5.2-2025-12-11` |
| `SCENE_NAME` | Default scene name | `room0` |

For a new machine, just update `REPLICA_ROOT` in `env_vars.bash` and run `source env_vars.bash`.

## Pipeline Commands

The pipeline runs from the `conceptgraph/` directory. Shell scripts in `bashes/` wrap the Python commands.

All scripts accept `SCENE_NAME` as first argument (default: `room0`):
```bash
bash bashes/1b_extract_2d_segmentation_detect.sh room0
```

### Pipeline Overview

There are two modes: **Class-Agnostic** (suffix `none`) and **Class-Aware/Detect** (suffix `b`, uses RAM + Grounding-DINO).

| Step | Class-Agnostic | Class-Aware (Detect) | Description |
|------|----------------|---------------------|-------------|
| 1 | `1_extract_2d_segmentation.sh` | `1b_extract_2d_segmentation_detect.sh` | 2D segmentation + CLIP features |
| 2 | `2_build_3d_object_map.sh` | `2b_build_3d_object_map_detect.sh` | 3D object map construction |
| 2.5 | - | `2.5b_fuse_lseg_dense_pointcloud.sh` | Dense point cloud + LSeg features |
| 3 | `3_visualize_object_map.sh` | - | Interactive Open3D visualization |
| 4 | `4_extract_object_captions.sh` | `4b_extract_object_captions_detect.sh` | VLM object captioning |
| 4.5 | `4.5_semantic_scene_segmentation.sh` | - | Region-aware scene segmentation |
| 5 | `5_refine_object_captions.sh` | `5b_refine_object_captions_detect.sh` | LLM caption refinement |
| 5+ | - | `5b_refine_with_affordance.sh` | Refinement + affordance extraction |
| 5.5 | - | `5.5b_hierarchical_segmentation.sh` | Hierarchical zone segmentation |
| 6 | `6_build_scene_graph.sh` | `6b_build_visibility_index.sh` | Scene graph / visibility index |
| 7 | `7_visualize_scene_graph.sh` | `7b_query_scene.sh` | Visualization / query demo |
| 7 | `7_visualize_scene_graph_offscreen.sh` | - | Offline visualization (headless) |

### Detailed Script Reference

#### Step 1: 2D Segmentation

**`1_extract_2d_segmentation.sh`** - Class-agnostic mode (SAM only)
- Input: RGB-D images in `$REPLICA_ROOT/$SCENE/results/`
- Output: `gsa_results_none/*.pkl.gz`, `gsa_vis_none/*.jpg`
- Mode: Pure SAM segmentation without class detection

**`1b_extract_2d_segmentation_detect.sh`** - Class-aware mode (RAM + Grounding-DINO + SAM)
- Input: RGB-D images
- Output: `gsa_results_ram_withbg_allclasses/*.pkl.gz`
- Mode: Detect objects with RAM tags, segment with SAM

#### Step 2: 3D Object Map

**`2_build_3d_object_map.sh`** - Build from class-agnostic results
- Input: `gsa_results_none/`, camera poses (`traj.txt`)
- Output: `pcd_saves/full_pcd_none_*_post.pkl.gz`
- Also generates PLY visualization in `ply_export_none/`

**`2b_build_3d_object_map_detect.sh`** - Build from class-aware results
- Input: `gsa_detections_ram_withbg_allclasses/`
- Output: `pcd_saves/full_pcd_ram_withbg_allclasses_*_post.pkl.gz`
- Also generates PLY visualization in `ply_export_detect/`

**`2.5b_fuse_lseg_dense_pointcloud.sh`** - Dense point cloud with LSeg features
- Input: RGB-D + poses
- Output: `dense_pcd_lseg.npz` (points + 512-dim LSeg features)
- Requires `lseg` conda environment

#### Step 3: Visualization

**`3_visualize_object_map.sh`** - Interactive Open3D viewer
- Input: `pcd_saves/*.pkl.gz`
- Hotkeys: `[b]` background, `[c]` class color, `[r]` RGB, `[f]` CLIP query, `[i]` instance ID

#### Step 4: Object Captioning

**`4_extract_object_captions.sh`** / **`4b_extract_object_captions_detect.sh`**
- Input: 3D object map
- Output: `sg_cache/cfslam_llava_captions.json`, CLIP features, debug images
- Uses VLM to generate multi-view object descriptions

**`4.5_semantic_scene_segmentation.sh`** - Region-aware scene segmentation
- Input: Object map + captions + trajectory
- Output: `sg_cache/segmentation_regions/` (regions.json, GIFs, keyframes)
- Detects functional zones (living room, kitchen, etc.)

#### Step 5: Caption Refinement

**`5_refine_object_captions.sh`** / **`5b_refine_object_captions_detect.sh`**
- Input: Raw captions from Step 4
- Output: `sg_cache/cfslam_*_responses/` refined descriptions
- LLM consolidates multi-view descriptions into unified labels

**`5b_refine_with_affordance.sh`** - Combined refinement + affordance
- Input: Captions + object images
- Output: `sg_cache_detect/object_affordances.json`
- Extracts object tags, summaries, categories, and affordances
- Args: `SCENE_NAME [IMAGE_NUM=4] [MAX_WORKERS=10]`

**`5.5b_hierarchical_segmentation.sh`** - Hierarchical zone segmentation
- Input: Object affordances from 5b+
- Output: `hierarchical_segmentation_detect/hierarchical_scene_graph.json`
- Creates zone hierarchy with 3D visualization

#### Step 6: Scene Graph / Index Building

**`6_build_scene_graph.sh`** - Build semantic scene graph
- Input: Refined captions
- Output: `sg_cache/map/scene_map_cfslam_pruned.pkl.gz`, `cfslam_object_relations.json`
- Uses LLM to infer spatial relations (on, in, near, etc.)

**`6b_build_visibility_index.sh`** - Build visibility index for queries
- Input: 3D object map
- Output: `indices/visibility_index.pkl`
- Precomputes object-view visibility for fast keyframe selection

#### Step 7: Visualization / Query

**`7_visualize_scene_graph.sh`** - Interactive scene graph viewer
- Input: Scene graph map + relations
- Hotkeys: `[g]` toggle graph, `[b]` background, `[+/-]` point size

**`7_visualize_scene_graph_offscreen.sh`** - Headless visualization
- Input: Scene graph map
- Output: `visualization/` (images, PLY, HTML)
- For servers without display

**`7b_query_scene.sh`** - Query-driven keyframe selection
- Usage: `./7b_query_scene.sh room0 "pillow on the sofa" 3`
- Args: `SCENE_NAME QUERY [K=3]`
- Output: `query_results/` with selected keyframes

#### Testing

**`run_e2e_query_test.sh`** - Run end-to-end query tests
- Logs to `docs/e2e_query_test_run.log`

### Recommended Pipelines

**Class-Aware (Detect) Pipeline** - Best for object-level queries:
```bash
bash bashes/1b_extract_2d_segmentation_detect.sh room0
bash bashes/2b_build_3d_object_map_detect.sh room0
bash bashes/4b_extract_object_captions_detect.sh room0
bash bashes/5b_refine_with_affordance.sh room0
bash bashes/5.5b_hierarchical_segmentation.sh room0
bash bashes/6b_build_visibility_index.sh room0
bash bashes/7b_query_scene.sh room0 "pillow on the sofa"
```

**Class-Agnostic Pipeline** - For scene graph construction:
```bash
bash bashes/1_extract_2d_segmentation.sh room0
bash bashes/2_build_3d_object_map.sh room0
bash bashes/4_extract_object_captions.sh room0
bash bashes/5_refine_object_captions.sh room0
bash bashes/6_build_scene_graph.sh room0
bash bashes/7_visualize_scene_graph.sh room0
```

### Query System Testing

```bash
# Run end-to-end query tests
bash bashes/run_e2e_query_test.sh

# Or directly
python conceptgraph/query_scene/examples/e2e_query_test.py
```

## Architecture

### Core Modules (`conceptgraph/`)

- **slam/**: 3D object mapping pipeline
  - `cfslam_pipeline_batch.py`: Main mapping script (Hydra config)
  - `slam_classes.py`: `DetectionList`, `MapObjectList` data structures
  - `mapping.py`: Spatial/visual similarity computation and object merging

- **query_scene/**: Natural language spatial query system
  - `query_parser.py`: LLM-based query parsing to `GroundingQuery` structures
  - `query_executor.py`: Bottom-up recursive query execution
  - `query_pipeline.py`: `QueryScenePipeline` orchestrates parsing/execution
  - `spatial_relations.py`: Geometric relation checking (on, near, above, etc.)
  - `scene_representation.py`: `QuerySceneRepresentation` wraps scene data

- **scenegraph/**: Scene graph construction
  - `build_scenegraph_cfslam.py`: Caption extraction and edge building with LLaVA/GPT

- **dataset/**: Data loading
  - `datasets_common.py`: `get_dataset()` returns dataset loaders (Replica, TUM, ScanNet)

- **llava/**: VLM integration
  - `unified_client.py`, `ollama_adapter.py`: Local VLM inference

- **utils/**: Shared utilities
  - `llm_client.py`: LangChain Azure OpenAI wrapper with model configs

### Key Data Flow

1. RGB-D images + poses → `generate_gsa_results.py` → 2D masks + CLIP features
2. 2D detections → `cfslam_pipeline_batch.py` → 3D objects (`.pkl.gz`)
3. 3D objects → `build_scenegraph_cfslam.py` → Scene graph with captions
4. Scene graph + query → `QueryScenePipeline` → Grounded objects

### Configuration

- Hydra configs in `conceptgraph/hydra_configs/`
- Default SLAM config: `hydra_configs/cfslam_pipeline_batch.yaml`

## Code Patterns

### Loading Scene Data

```python
import gzip, pickle
with gzip.open("path/to/scene_post.pkl.gz", "rb") as f:
    data = pickle.load(f)
objects = data.get("objects", [])  # List of dicts with 'pcd_np', 'clip_ft', 'class_name', etc.
```

### Query Pipeline Usage

```python
from conceptgraph.query_scene.query_pipeline import QueryScenePipeline

pipeline = QueryScenePipeline.from_scene(
    scene_path="/path/to/room0",
    llm_model="gpt-5.2-2025-12-11"
)
result = pipeline.query("the pillow on the sofa near the door")
```

### Spatial Relations

```python
from conceptgraph.query_scene.spatial_relations import SpatialRelationChecker

checker = SpatialRelationChecker(objects)
result = checker.check("on", obj_a, obj_b)  # Returns RelationResult with score
```

## Output Formats

- **3D object map**: `.pkl.gz` containing `{"objects": [...], "bg_objects": [...], ...}`
- **Scene graph**: JSON with node captions and edge relations
- **Visualization**: PLY point clouds in `pcd_saves/ply_export_detect/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yruns)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yruns)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
