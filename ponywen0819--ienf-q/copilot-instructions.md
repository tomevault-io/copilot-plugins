## ienf-q

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

IENF-Q (Intra-Epidermal Nerve Fiber Quantification) is an automated analysis pipeline that reconstructs complete neural fiber networks from microscopic images and sparse manual annotations. It uses classical computer vision algorithms (no ML) for interpretable, reproducible results.

## Build and Run Commands

```bash
# Install dependencies (uses uv package manager)
uv sync

# Run complete pipeline (preprocessing + reconstruction)
python test_pipeline.py

# Run preprocessing only
python tools/run_preprocessing.py \
    --label data/Label/S163-2_a.tif \
    --mask data/Mask/S163-2_a.tif \
    --image data/Original/S163-2_a.tif \
    --output-dir output/preprocessing \
    --debug

# Run reconstruction only (legacy)
python test_main_entry.py

# Run with uv
uv run python test_pipeline.py

# Run tests
pytest test/
pytest test/construction/component_analyzer/  # Run specific test module
```

## Architecture

The codebase is organized into a layered architecture with clear separation of concerns:

### Project Structure

```
src/neural_reconstruction/
├── common/              # Shared data types and utilities
│   └── data_types.py   # ComponentAnalysisResult, ConnectionGraphBuilderResult
├── core/               # Core algorithms
│   ├── preprocessing/  # Image preprocessing pipeline
│   ├── construction/   # Neural network reconstruction
│   │   ├── component_analyzer/     # Skeletonization, topology, seed extraction
│   │   ├── connection_graph_builder/  # A* path finding, graph building
│   │   ├── backbone_extractor/     # MST extraction
│   │   └── main.py                 # build_neural_network() entry point
│   └── crosses_detection/  # Epidermis crossing detection
└── ui/                 # Main pipeline integration
    └── main_pipeline.py  # NeuralReconstructionPipeline class
```

### Complete Pipeline: Preprocessing + Reconstruction

**NEW: Unified Pipeline** - Use `NeuralReconstructionPipeline` in [src/neural_reconstruction/ui/main_pipeline.py](src/neural_reconstruction/ui/main_pipeline.py) for end-to-end processing from raw images to reconstructed network. This integrates preprocessing and reconstruction into a single API.

**Lower-level API** - `build_neural_network()` in [src/neural_reconstruction/core/construction/main.py](src/neural_reconstruction/core/construction/main.py) provides direct access to the reconstruction algorithm, assuming preprocessing is already done.

### Neural Network Reconstruction Process

The reconstruction process chains together four distinct phases:

**Phase 1: Connected Components Analysis**
- Uses scikit-image's `label()` and `regionprops()`
- Identifies discrete fiber segments from binary annotations
- Filters by minimum area to remove noise

**Phase 2: Component Analysis** (`component_analyzer/`)
- **Skeletonization**: Extracts centerlines using Zhang-Suen algorithm
- **Topology Construction**: Builds graph representation using `skan` library
- **Seed Extraction**: Curvature-aware placement along skeleton
  - More seeds at bends/branches to preserve fiber geometry
  - Controlled by `segment_length` parameter

**Phase 3: Connection Graph Building** (`connection_graph_builder/`)
- **Path Finding**: A* algorithm finds connections between components
- **Cost Calculation**: Multi-factor cost function:
  - `intensity_weight`: Prefers paths along bright pixels (nerve tissue)
  - `shape_weight`: Considers path smoothness and geometry
- **Graph Assembly**: Creates `NetworkX` graph with all feasible connections

**Phase 4: Backbone Extraction** (`backbone_extractor/`)
- **MST Construction**: Kruskal's algorithm for minimum spanning tree
- **Forest Output**: Produces optimal fiber network (may have multiple trees)

### Preprocessing Pipeline

Located in [src/neural_reconstruction/core/preprocessing/pipeline.py](src/neural_reconstruction/core/preprocessing/pipeline.py), the `SkinAnalysisPipeline` class handles:

1. **Green Channel Extraction**: Neural tissue has strongest signal in green channel
2. **Morphological Operations**: Opening/closing to clean binary masks
3. **Background Correction**: Rolling ball algorithm (radius configurable)
4. **Sato Vesselness Filter** (optional): Enhances tubular/fiber structures for better detection
5. **Multi-Otsu Thresholding**: More robust than single Otsu for complex intensity distributions
6. **Mask Operations**: ROI extraction using epidermis mask with configurable dilation

### Crosses Detection Module

The `crosses_detection/` module counts nerve fibers crossing the epidermis boundary:
- `segment_detector.py`: Identifies fiber segments
- `region_labeler.py`: Labels epidermis regions
- `crossing_counter.py`: Counts crossings for quantification

## Key API Entry Points

### Complete Pipeline (Recommended)

```python
from neural_reconstruction.ui.main_pipeline import NeuralReconstructionPipeline

# Using default configuration
pipeline = NeuralReconstructionPipeline()

# From file paths
result = pipeline.run_from_files(
    label_path="data/Label/S163-2_a.tif",
    mask_path="data/Mask/S163-2_a.tif",
    image_path="data/Original/S163-2_a.tif"
)

# From NumPy arrays
result = pipeline.run(
    label_image=label_array,      # Binary annotation (H, W)
    mask_image=mask_array,        # Epidermis mask (H, W)
    original_image=original_array # RGB or grayscale (H, W, 3) or (H, W)
)

# Access results
mst_forest = result.mst_forest        # NetworkX Graph
final_label = result.final_label      # Processed label (H, W)
roi_image = result.roi_image          # ROI image (H, W)
print(f"Nodes: {result.num_nodes}, Edges: {result.num_edges}")
```

### Custom Configuration

```python
# Custom preprocessing config
preprocessing_config = {
    'morphology': {'closing_kernel': 5, 'opening_kernel': 3},
    'mask': {'dilate_offset': 100},
    'background': {
        'method': 'rolling_ball',
        'radius': 2,
        'sato_weight': 0.3,  # >0 enables Sato filter
        'sato_sigmas': (1, 2)
    },
    'threshold': {'use_full_roi': False},
    'normalization': {'enabled': True}
}

# Custom reconstruction config
reconstruction_config = {
    'connectivity': 4,
    'min_area': 30,
    'segment_length': 3.0,
    'search_radius': 100.0,
    'max_cost_threshold': 0.95,
    'intensity_weight': 0.7,
    'shape_weight': 0.3
}

pipeline = NeuralReconstructionPipeline(
    preprocessing_config=preprocessing_config,
    reconstruction_config=reconstruction_config
)
```

### Reconstruction Only (Lower-Level)

```python
from neural_reconstruction.core.construction.main import build_neural_network

# Minimal usage
mst_forest = build_neural_network(
    label_image=binary_label,      # np.ndarray (H, W) binary
    green_channel=green_channel,   # np.ndarray (H, W) uint8
)

# With all parameters
mst_forest = build_neural_network(
    label_image=binary_label,
    green_channel=green_channel,
    connectivity=4,                # 4 or 8
    min_area=50,                  # Filter small components
    segment_length=5.0,           # Seed spacing (pixels)
    search_radius=50.0,           # Max connection distance
    max_cost_threshold=0.98,      # Path cost cutoff (0-1)
    intensity_weight=0.6,         # Path cost weights
    shape_weight=0.4,
)
# Returns: NetworkX Graph with nodes=(y,x) coords, edges with 'path' attribute
```

### Preprocessing Pipeline

```python
from neural_reconstruction.core.preprocessing.pipeline import SkinAnalysisPipeline

config = {
    'morphology': {'closing_kernel': 3, 'opening_kernel': 3},
    'background': {'radius': 12, 'light_background': True},
    'mask': {'dilate_offset': 50}
}

pipeline = SkinAnalysisPipeline(config)
final_label, roi_image = pipeline.run(
    label_img,      # Binary annotation
    mask_img,       # Epidermis mask
    orig_img,       # Original RGB/grayscale
    debug=False
)
```

## Key Configuration Parameters

### Preprocessing Configuration
- `morphology.closing_kernel`: Kernel size for morphological closing (default: 3)
- `morphology.opening_kernel`: Kernel size for morphological opening (default: 3)
- `mask.dilate_offset`: Vertical dilation offset in pixels (default: 100)
- `background.method`: 'morphology', 'rolling_ball', or 'gaussian' (default: 'rolling_ball')
- `background.radius`: Rolling ball radius (default: 2)
- `background.sato_weight`: Sato filter blend weight, 0=disabled, >0 enables with blending up to 1 (default: 0.0)
- `background.sato_sigmas`: Scale range tuple (min, max) for Sato filter to detect fiber widths (default: (1, 2))
- `threshold.use_full_roi`: Use full ROI image for pseudo-label generation instead of masked region (default: False)
  - When `False`: Only dermis ROI mask region is used for multi-Otsu thresholding
  - When `True`: Entire ROI image is used for multi-Otsu thresholding (may capture more fiber details)
  - Note: Uses **multi-level Otsu thresholding** which is more robust for complex intensity distributions
- `normalization.enabled`: Enable regional normalization (default: True)

### Reconstruction Configuration

**Component Analysis:**
- `connectivity`: 4 or 8 (default: 4) - affects connected component detection
- `min_area`: Minimum pixels to keep component (default: 50)
- `segment_length`: Seed spacing along skeleton in pixels (default: 5.0)
- `prune_threshold`: Remove skeleton branches shorter than this (default: 5.0)

**Path Finding:**
- `search_radius`: Max distance to search for connections in pixels (default: 50.0)
- `max_cost_threshold`: Reject paths with normalized cost > this value (default: 0.98)
- `intensity_weight`: Weight for image intensity in cost (default: 0.6)
- `shape_weight`: Weight for path geometry in cost (default: 0.4)

## Code Conventions

- **Language**: Python 3.10+, code comments in Traditional Chinese
- **Module naming**: The package is `neural_reconstruction` (note: some legacy references may say `nueral_reconstruction` - when updating code, use `neural_reconstruction`)
- **Image format**:
  - Green channel carries strongest nerve fiber signal (automatically extracted from RGB)
  - Annotations are binary (255=fiber, 0=background)
  - Internal processing uses 0/1 binary after conversion
  - Original images can be RGB (H, W, 3) or grayscale (H, W) - pipeline handles both
- **Data structures**:
  - Uses `NetworkX` for graph representations
  - Custom dataclasses defined in `common/data_types.py`
  - NumPy arrays for all image data
- **Connectivity Parameter**: Note that scikit-image uses 1 for 4-connected and 2 for 8-connected, while our API uses 4 and 8. The conversion is handled internally in `build_neural_network()`

## Package Management

This project uses **uv** as the package manager:
- Dependencies defined in [pyproject.toml](pyproject.toml)
- Requires Python >=3.10
- Key dependencies: opencv-python, scikit-image, networkx, skan, pyyaml

## Development Notes

- **Entry points**: Main development entry points are:
  - [test_pipeline.py](test_pipeline.py) - Tests complete pipeline (preprocessing + reconstruction)
  - [examples/pipeline_usage.py](examples/pipeline_usage.py) - Usage examples with different configurations
  - [test_main_entry.py](test_main_entry.py) - Tests reconstruction only (legacy)
  - [tools/run_preprocessing.py](tools/run_preprocessing.py) - CLI for preprocessing only
- **Data directory**: [data/](data/) contains sample images for testing (Original/, Label/, Mask/ subdirectories)
- **Test structure**: Tests are organized in [test/](test/) directory with pytest
  - `test/construction/component_analyzer/` - Component analysis tests
  - `test/construction/connection_graph_builder/` - Path finding and graph building tests
- **Output**: Pipeline generates NetworkX graphs; visualization is handled externally
- **Edge case handling**: The pipeline handles graphs with no edges (isolated components) gracefully. This is expected when components are too far apart or cost threshold is too strict.

## Evaluation and Topology Tools

The project includes specialized tools for topology extraction, comparison, and evaluation:

### Dataset GT Topology Extraction

Extract Ground Truth topologies from all samples with `label.png`:

```bash
# Extract all GT topologies from data/ directory
uv run python tools/extract_dataset_topologies.py

# Custom paths
uv run python tools/extract_dataset_topologies.py \
    --data-dir data \
    --output-dir output/gt_topologies \
    --verbose
```

**Features**:
- Automatically scans `data/` for samples with `label.png`
- Extracts complete topology using `TopologyExtractor`
- Saves as `.pkl` files (e.g., `S1585-2_a_gt.pkl`)
- Provides detailed extraction statistics

**Documentation**: [docs/DATASET_TOPOLOGY_EXTRACTION.md](docs/DATASET_TOPOLOGY_EXTRACTION.md)

### Topology Comparison

Compare topologies without running the full image processing pipeline:

```bash
# Compare two topologies
uv run python tools/compare_topologies.py \
    --topology1 output/pred.pkl \
    --topology2 output/gt.pkl

# Batch comparison
uv run python tools/compare_topologies.py \
    --batch \
    --pred-dir output/predictions \
    --gt-dir output/gt_topologies \
    --output results.csv
```

**Features**:
- Supports Pickle, JSON, GraphML, GML formats
- Computes Average Hausdorff Distance
- Includes both nodes and edge path points
- Single-pair and batch comparison modes

**Documentation**: [docs/TOPOLOGY_COMPARISON.md](docs/TOPOLOGY_COMPARISON.md)

### Complete Evaluation

Evaluate full pipeline including preprocessing and reconstruction:

```bash
uv run python tools/evaluate_dataset.py \
    --data-dir data \
    --output-dir output/evaluation \
    --sample-ids S1585-2_a S1585-2_b
```

**Features**:
- End-to-end evaluation from images to metrics
- Average Hausdorff Distance with edge path points
- Batch processing with statistical summaries
- JSON and CSV output formats

### Quick Reference

All tools are documented in [QUICK_REFERENCE.md](QUICK_REFERENCE.md) with common commands and workflows.

### Evaluation Metrics

- **Average Hausdorff Distance**: `d(A→B) = mean(min_distance(a, B))`, symmetric average
- **Point Sets**: Includes graph nodes + all edge path points (`path` or `path-coordinates` attributes)
- **More Robust**: Less sensitive to outliers than maximum Hausdorff distance

---
> Source: [Ponywen0819/ienf_q](https://github.com/Ponywen0819/ienf_q) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
