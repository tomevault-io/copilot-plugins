## scene-optimizer-presets

> How to compose Scene Optimizer preset JSON files. Covers operation catalog, ordering, Python processors, and parameter guidance.


# Scene Optimizer Preset Composition

Scene Optimizer presets are JSON arrays of operation objects that define a processing pipeline for USD assets. This guide covers how to compose them.

## Agent Behavior

When a user asks to create or modify a preset:

1. **Start from the canonical template** - read `so/generic/generic_preset.json` as a baseline rather than building from scratch.
2. **Ask what problems they are solving** - map their issues to operations using the catalog below.
3. **Follow operation ordering strictly** - the ordering rules in this file prevent data corruption (e.g., running `deduplicateGeometry` before splitting GeomSubsets produces wrong results silently).
4. **Check vendor presets first** - for vendor equipment, look in `so/vertiv/`, `so/spt/`, `so/trane/` for existing presets before creating a new one.
5. **Prefer external scripts** - when adding `pythonScript` operations, use the library pattern (loading from `so/generic/lib/`) over base64-embedded scripts for maintainability.

When a user asks to fix a specific validation failure with a preset:

1. Look up the fix operation in `.cursor/rules/usd-universal.mdc` (Quick Rule-to-Operation Lookup table).
2. Create a minimal preset with just the needed operations, respecting the ordering rules below.
3. If only one or two operations are needed, a targeted preset is better than running the full generic preset.

## Preset Structure

A preset is a JSON array where each element is an operation:

```json
[
    { "operation": "editStageMetrics", "metersPerUnit": 1.0, "upAxis": 2 },
    { "operation": "meshCleanup", "paths": [], "mergeVertices": true },
    { "operation": "generateNormals", "paths": [], "sharpnessAngle": 60.0 }
]
```

Each operation object must have an `"operation"` key. Additional keys are operation-specific parameters. The full parameter reference is in `so_operations.json` (97KB).

## Operation Catalog

### Stage Operations

| Operation | Purpose | Key Parameters |
|-----------|---------|----------------|
| `editStageMetrics` | Set units, up-axis, collapse xforms | `metersPerUnit`, `upAxis` (2=Z), `collapseXforms`, `ignoreKitCameras` |

### Geometry Cleanup

| Operation | Purpose | Key Parameters |
|-----------|---------|----------------|
| `meshCleanup` | Fix topology issues | `mergeVertices`, `tolerance`, `removeDegenerateFaces`, `removeDuplicateFaces`, `removeIsolatedVertices`, `makeManifold`, `contractDegenerateEdges`, `mergeBoundaries`, `mergeNeighbors` |
| `generateNormals` | Generate surface normals | `sharpnessAngle` (degrees), `replaceExisting`, `binding` (0=vertex), `weightmode`, `gpuThreshold` |
| `computeExtents` | Compute bounding extents | `paths` |
| `removeSmallGeometry` | Remove tiny geometry | `removeMethod`, `detectionMethod`, `threshold` |
| `manifoldMeshes` | Make meshes watertight | `paths` |

### Geometry Optimization

| Operation | Purpose | Key Parameters |
|-----------|---------|----------------|
| `decimateMeshes` | Reduce polygon count | `reductionFactor`, `maxMeanError`, `pinBoundaries`, `allowCutAndGlue`, `cpuVertexCountThreshold`, `gpuVertexCountThreshold`, `guideDecimation` |
| `deduplicateGeometry` | Instance identical meshes | `tolerance`, `duplicateMethod` (2=hierarchy), `fuzzy`, `allowScaling`, `considerDeepTransforms`, `useGpu`, `ignoreAttributes` |
| `merge` | Merge meshes into one | `paths` |
| `remeshMeshes` | Remesh geometry | `paths` |
| `triangulateMeshes` | Convert to triangles | `paths` |
| `subdivideMeshes` | Subdivide surfaces | `paths` |
| `diceMeshes` | Subdivide/dice geometry | `paths` |
| `splitMeshes` | Split meshes by criteria | `paths` |
| `boxClip` | Clip meshes by box | `paths` |

### Hierarchy Operations

| Operation | Purpose | Key Parameters |
|-----------|---------|----------------|
| `pruneLeaves` | Remove empty leaf nodes | `pruneMode` (1=empty), `filterInactive` |
| `flattenHierarchy` | Flatten prim hierarchy | `paths` |
| `findFlatHierarchies` | Detect flat hierarchies | `paths` |
| `pivot` | Adjust pivot points | `paths` |

### Material Operations

| Operation | Purpose | Key Parameters |
|-----------|---------|----------------|
| `optimizeMaterials` | Deduplicate/consolidate materials | `optimizeMaterialsMode` (0=consolidate, 2=deduplicate), `materialsPath`, `analysisCheckPrimvars` |
| `optimizePrimvars` | Clean up primvar data | `mode`, `simplify`, `removeIfBound`, `primvars`, `primvarPaths` |

### Analysis Operations (non-destructive)

| Operation | Purpose |
|-----------|---------|
| `findCoincidingMeshes` | Identify overlapping meshes |
| `findHiddenMeshes` | Locate hidden geometry |
| `fitPrimitives` | Fit primitive shapes to meshes |

### Other Operations

| Operation | Purpose |
|-----------|---------|
| `pythonScript` | Run custom Python code |
| `removeAttributes` | Remove prim attributes |
| `generateAtlasUVs` | Generate texture atlas UVs |
| `generateProjectionUVs` | Generate projected UVs |
| `organizePrototypes` | Organize instanced prototypes |
| `optimizeSkelRoots` | Optimize skeleton roots |
| `optimizeTimeSamples` | Reduce time samples |
| `primitivesToMeshes` | Convert primitives to meshes |
| `utilityFunction` | Execute utility functions |

## Recommended Operation Ordering

Based on the canonical `so/generic/generic_preset.json`:

```
1. editStageMetrics                    ← Always first: normalize units/orientation
2. pythonScript (split GeomSubsets)    ← Split before dedup (dedup doesn't support GeomSubsets)
3. pythonScript (hierarchy dedup)      ← Dedup branches before mesh-level ops
4. meshCleanup                         ← Fix topology before decimation
5. decimateMeshes                      ← Reduce poly count
6. generateNormals                     ← Regenerate after geometry changes
7. optimizeMaterials (mode 2)          ← Deduplicate materials
8. deduplicateGeometry                 ← Instance identical meshes
9. pruneLeaves                         ← Clean up empty nodes from dedup
10. optimizePrimvars                   ← Remove unnecessary data
11. optimizeMaterials (mode 0)         ← Final material consolidation
12. pythonScript (fix MaterialBinding) ← Fix dangling bindings and missing API schema
13. removeSmallGeometry                ← Remove degenerate geometry
14. pruneLeaves                        ← Second pass cleanup
15. computeExtents                     ← Always last: recompute after all changes
```

**Key ordering rules:**
- Stage metrics MUST be first
- GeomSubset split MUST precede dedup (dedup doesn't support GeomSubsets yet)
- Cleanup MUST precede decimation
- Normals MUST follow decimation (geometry changed)
- Extents MUST be last (depends on final geometry)
- Prune after dedup and after small geometry removal (two passes)
- Material dedup before geometry dedup
- MaterialBindingAPI fix after all material operations

## Python Script Processors

### Embedded Script (base64-encoded)

```json
{
    "operation": "pythonScript",
    "python": "<base64-encoded Python code>"
}
```

The `python` value is base64-encoded. To encode:
```python
import base64
code = open("my_script.py").read()
encoded = base64.b64encode(code.encode()).decode()
```

### External Script Pattern (recommended for development)

Store scripts in a library folder and load at runtime:

```python
import os

lib_path = os.path.join(os.environ["AIF_PIPELINE_SAMPLES_ROOT"], "so", "generic", "lib")
script_file = os.path.join(lib_path, "my_script.py")

exec(compile(open(script_file).read(), script_file, 'exec'))
my_function()
```

This reads fresh from disk (no Kit restart needed) and provides proper error tracebacks.

**Warning:** `if __name__ == "__main__":` does NOT protect against auto-execution in `exec()` context. Either remove the block or pass a custom name.

### Available Library Scripts (`so/generic/lib/`)

| Script | Entry Point | Purpose | When to Use |
|--------|------------|---------|-------------|
| `add_layers.py` | `add_layers_from_folder()` | Auto-discovers `./layers/` subfolder, composes as sublayers, updates version metadata | Final step: compose metadata and connection point layers |
| `deduplicate_hierarchies_by_display_name.py` | `process_duplicate_hierarchies(use_payloads=False, merge_prototype_hierarchies=False)` | Groups prims by display name, converts duplicates to instanceable refs or payloads | Early: before mesh-level dedup. Two-phase approach for large assemblies |
| `group.py` | `group_prims(group_name, paths)` | Reparents prims under new Xform preserving world-space transforms (uses `Sdf.BatchNamespaceEdit`) | Hierarchy reorganization |
| `material_replacement.py` | `replace_materials()` | Replaces CAD materials with UsdPreviewSurface. 7 built-in materials (slate_gray, plastic, shiny_plastic, logo_white, glass, steel, screen). Configure via `MATERIAL_REPLACEMENT_TEMPLATES`, `FALLBACK_MATERIAL`, `PRIM_PATH_MATERIAL_OVERRIDES` | After CAD import; standardize materials |
| `remove_coinciding_meshes.py` | `remove_coinciding_meshes(debug=False)` | Uses SO `findCoincidingMeshes` then removes duplicates (tolerance=0.001) | When meshes overlap at same location (z-fighting) |
| `set_forward.py` | `set_x_forward()` | Applies 90° Y-rotation, decomposes matrix into T-R-S ops | CAD assets facing wrong direction |
| `split_non_composed_by_geom_subsets.py` | `split_all_meshes()` | Splits non-composed meshes by GeomSubsets, skips refs/payloads | **Must run before dedup** (dedup doesn't support GeomSubsets) |
| `transform_stage.py` | `transform_stage(translate, rotate, scale, up_axis)` | Unified transform interface. Convenience: `set_z_up()`, `rotate_x/y/z(degrees)`, `identity_transform()` | Reposition, rotate, scale assets |
| `validate_fix_material_binding_api.py` | `validate_and_fix_material_binding_apis()` | Removes dangling bindings (checks `material:binding`, `:collection`, `:preview`, `:full`), applies missing MaterialBindingAPI schema | After material operations; cleanup pass |

### Script Environment

Inside a pythonScript processor, you have access to:
- `omni.usd.get_context().get_stage()` — the current USD stage
- `from pxr import Usd, Sdf, UsdGeom, Gf` — OpenUSD Python bindings
- `from omni.scene.optimizer.core import ExecutionContext` — SO execution context
- `stage.GetRootLayer().customLayerData` — store data for cross-operation coordination
- `Sdf.BatchNamespaceEdit()` — atomic prim reparenting

## Parameter Guidance by Scenario

### Aggressive Optimization (AIF real-time factory scenes)

```json
{
    "operation": "decimateMeshes",
    "paths": [],
    "reductionFactor": 0.0,
    "maxMeanError": 0.0001,
    "pinBoundaries": true,
    "gpuVertexCountThreshold": 500000
}
```

- Use error-based decimation (`maxMeanError`) rather than fixed ratio
- Pin boundaries to preserve connection surfaces
- Enable GPU acceleration for large meshes

### Conservative Optimization (visual quality preservation)

```json
{
    "operation": "decimateMeshes",
    "paths": [],
    "reductionFactor": 0.0,
    "maxMeanError": 0.00005,
    "pinBoundaries": true,
    "allowCutAndGlue": false
}
```

### Paths Parameter

- `"paths": []` — apply to all meshes in the stage
- `"paths": ["/World/Equipment/Panel"]` — apply only to specific prims
- Use targeted paths when different parts need different treatment

### Material Modes

- `"optimizeMaterialsMode": 0` — consolidate (merge compatible materials)
- `"optimizeMaterialsMode": 2` — deduplicate (remove exact duplicates)
- Run dedup (2) first, consolidate (0) after geometry operations

## Running Presets

```bash
# Via CLI (recommended)
aif-pipeline optimize input/ output/ --preset path/to/preset.json

# Via direct script
python scripts/optimize.py input/ output/ --preset path/to/preset.json --kit_path "D:/kit/kit.exe"

# In USD Composer GUI
# Window > Utilities > Scene Optimizer > Load Preset > Execute All
```

## Reference Presets

Study these for patterns:
- `so/generic/generic_preset.json` — canonical multi-step pipeline
- `so/spt/gb300/gb300.json` — GB300 rack with custom Python processors
- `so/trane/220LL.json` — vendor-specific with stage preparation and material replacement
- `so/vertiv/<model>/<model>.json` — payload assembly patterns

---
> Source: [NVIDIA-Omniverse/aif-pipeline-samples](https://github.com/NVIDIA-Omniverse/aif-pipeline-samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
