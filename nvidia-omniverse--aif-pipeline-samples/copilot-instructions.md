## usd-universal

> USD quality rules for diagnosing and fixing asset issues. Covers core geometry/material quality, SimReady compliance, and the full validation rule catalog.


# USD Quality Rules

Three tiers: **Core Quality** (valid, clean USD), **SimReady Compliance** (standardized assets per the SimReady spec), and **AIF-specific** (covered in `.cursor/rules/usd-aif-profile.mdc`).

## Agent Behavior

When a user reports a USD quality issue or a validation rule failure:

1. **Identify the rule name** from their validation output (e.g., `NormalsExistChecker`, `ExtentsChecker`).
2. **Check the Quick Lookup** table below first for the most common failures.
3. **For less common rules,** scan the Symptom-to-Fix tables in the sections that follow.
4. **If multiple fixes are needed,** compose them into a preset respecting the operation ordering in `.cursor/rules/scene-optimizer-presets.mdc`.
5. **Re-validate** after applying fixes to confirm resolution.
6. If a rule is not in these tables, check `.cursor/rules/usd-issues-catalog.mdc` for known patterns.

### Quick Rule-to-Operation Lookup

For the most common validation failures, here are direct fixes:

| Failed Rule | Fix Operation | Key Parameters |
|---|---|---|
| `NormalsExistChecker` / `AIFNormalsValidChecker` | `generateNormals` | `sharpnessAngle: 60.0, replaceExisting: true` |
| `NormalsWindingsChecker` | `generateNormals` | `replaceExisting: true` |
| `ExtentsChecker` / `ZeroExtentChecker` | `computeExtents` | `paths: []` (all prims) |
| `UpAxisZChecker` / `AIFMetersPerUnitChecker` | `editStageMetrics` | `upAxis: 2, metersPerUnit: 1.0` |
| `ValidateTopologyChecker` | `meshCleanup` | `removeDegenerateFaces: true, contractDegenerateEdges: true` |
| `UsdDanglingMaterialBinding` | pythonScript | `validate_fix_material_binding_api.py` |
| `AIFAssetAtOriginChecker` | pythonScript | `transform_stage.py` with identity transform |
| `AIFHierarchyHasRootChecker` | pythonScript | `group.py` to create single root |
| `AIFMetadataChecker` | Not an SO fix | See metadata workflow in `.cursor/rules/usd-aif-profile.mdc` |
| `AIFGeomShallBeMeshChecker` | `primitivesToMeshes` | Convert non-mesh geometry to meshes |

## Symptom → Fix Reference

Use these tables to map a problem to its Scene Optimizer fix. For the full validation rule catalog, see the end of this file.

### Geometry Issues

| Symptom | Detection Rule | Fix (SO Operation) | Parameters |
|---|---|---|---|
| Degenerate/zero-area faces | `ZeroAreaFacesChecker`, `ValidateTopologyChecker` | `meshCleanup` | `removeDegenerateFaces: true, contractDegenerateEdges: true` |
| Non-manifold edges | `NonManifoldChecker` | `meshCleanup` | `makeManifold: true` |
| Duplicate faces | `DuplicateFaceChecker` | `meshCleanup` | `removeDuplicateFaces: true` |
| Isolated/unreferenced vertices | `IsolatedVerticesChecker`, `UnusedMeshTopologyChecker` | `meshCleanup` | `removeIsolatedVertices: true` |
| Colocated vertices | `ColocatedVerticesChecker` | `meshCleanup` | `mergeVertices: true, tolerance: 0.0` |
| Missing normals | `NormalsExistChecker` | `generateNormals` | `sharpnessAngle: 60.0, replaceExisting: true` |
| Invalid normals (NaN/Inf) | `AIFNormalsValidChecker` | `generateNormals` | `replaceExisting: true` |
| Inconsistent winding | `NormalsWindingsChecker`, `WindingsChecker` | `generateNormals` | `replaceExisting: true` |
| Zero/missing extents | `ExtentsChecker`, `ZeroExtentChecker` | `computeExtents` | `paths: []` (all prims) |
| High vertex count | `HighVertexCountChecker` | `decimateMeshes` | `maxMeanError: 0.0001, pinBoundaries: true` |
| Subdivision scheme undefined | `SubdivisionSchemeChecker` | Set explicitly | USD defaults to Catmull-Clark if not set |
| Non-indexed primvars | `IndexedPrimvarChecker` | `optimizePrimvars` | Use indexed format for storage optimization |
| Unused primvars/UVs | `UnusedPrimvarChecker`, `UnusedUVsChecker` | `optimizePrimvars` | `mode: 1, simplify: true` |
| Coinciding/overlapping meshes | `CoincidingGeometryChecker` | pythonScript: `remove_coinciding_meshes.py` | tolerance=0.001 |
| Occluded meshes | `OccludedMeshesChecker` | Review manually or remove | Optional GPU-accelerated analysis |
| Invisible prims | `InvisiblePrimsChecker` | Deactivate instead of hide | Invisible prims still consume resources |
| Sparse meshes needing splitting | `SparseMeshChecker` | `splitMeshes` or `diceMeshes` | Needs dicing, splitting, or clustering |
| Meshes replaceable by primitives | `PrimitiveFitChecker` | `fitPrimitives` | Replace with USD primitive prims |
| Points precision error | `PointsPrecisionErrorChecker` | Fix source data | Points must have precision for <1.0 unit increments |
| Extreme extents (>2^38) | `AlmostExtremeExtentChecker` | Fix transforms/scale | RTX limit is 2^40 |
| Meshes with GeomSubsets blocking dedup | Manual analysis | pythonScript: `split_non_composed_by_geom_subsets.py` | **Must run before dedup** |

### Material Issues

| Symptom | Detection Rule | Fix (SO Operation) | Parameters |
|---|---|---|---|
| Dangling material bindings | `UsdDanglingMaterialBinding` | pythonScript: `validate_fix_material_binding_api.py` | Checks 4 relationship types |
| Missing MaterialBindingAPI | `UsdMaterialBindingApi` | pythonScript: `validate_fix_material_binding_api.py` | Auto-applies API schema |
| Duplicate materials | `DuplicateMaterialsChecker` | `optimizeMaterials` | `optimizeMaterialsMode: 2` |
| Material outside payload hierarchy | `MaterialOutOfScopeChecker` | Restructure material paths | USD ignores bindings targeting outside payloads |
| Invalid MDL asset paths | `MaterialPathChecker` | Fix paths | MDL needs absolute or `./` relative paths |
| Deprecated MDL schema | `MaterialOldMdlSchemaChecker` | Update schema | `info:implementationSource` should not be `mdlMaterial` |
| Invalid UsdPreviewSurface | `OmniMaterialUsdPreviewSurfaceChecker` | Fix shader inputs | Must conform to UsdPreviewSurface spec |
| Missing shader implementation | `ShaderImplementationSourceChecker` | Add source | Needs `id`, `sourceAsset`, or `sourceCode` |
| Bad normal map texture setup | `NormalMapTextureChecker` | Fix UsdUVTexture | scale=[2,2,2,1], bias=[-1,-1,-1,0], sourceColorSpace=raw |
| Invalid GeomSubset material binding | `UsdGeomSubsetChecker` | Fix family name | Must have valid family name attribute |

### Hierarchy & Stage Issues

| Symptom | Detection Rule | Fix (SO Operation) | Parameters |
|---|---|---|---|
| No default prim | `OmniDefaultPrimChecker`, `DefaultPrimChecker` | Set via USD API | Single, active, Xformable or Scope root |
| Missing upAxis/metersPerUnit | `StageMetadataChecker` | `editStageMetrics` | Stages should declare both |
| Flat hierarchies (500+ children) | `FlatHierarchiesChecker` | pythonScript: `group.py` | Restructure into groups |
| Duplicate subtrees | `DuplicateGeometryChecker`, `FuzzyDuplicateGeometryChecker` | `deduplicateGeometry` | `tolerance: 0.001, duplicateMethod: 2, fuzzy: true` |
| Duplicate hierarchies | Manual analysis | pythonScript: `deduplicate_hierarchies_by_display_name.py` | `process_duplicate_hierarchies()` |
| Empty leaf prims | `EmptyLeafChecker` | `pruneLeaves` | `pruneMode: 1` |
| Orphaned prims (over specifier only) | `OmniOrphanedPrimChecker` | Fix or remove | Prims usually need `def` or `class` specifier |
| Untyped prims | `TypeChecker` | Add type | All prims must have a type defined |
| Non-portable asset paths (backslashes) | `PortableAssetPathChecker` | Fix paths | Use forward slashes |
| Non-anchored asset paths | `AnchoredAssetPathsChecker` | Fix paths | Use `./` or `../` relative paths |
| Unresolvable references | `MissingReferenceChecker` | Fix or remove refs | Stage should have no broken dependencies |
| Redundant time samples | `RedundantTimeSamplesChecker` | Remove duplicates | Analyzes for identical consecutive samples |
| Large arrays in .usda | `UsdAsciiPerformanceChecker` | Convert to .usdc | Large arrays/time samples belong in crate files |
| RTX mesh count exceeded | `RtxMeshCountChecker` | Merge or dedup | Check against RTX recommended limits |
| Unicode naming issues | `UnicodeNameChecker` | Normalize names | NFC normalized, no ambiguous children |
| Invalid kind values | `KindChecker` | Fix kind | Must be registered and conform to USD rules |
| Invalid layer specs | `LayerSpecChecker` | Fix types | Must have valid type names and value types |

### Performance Issues

| Symptom | Detection Rule | Fix (SO Operation) | Parameters |
|---|---|---|---|
| Duplicate geometry | `DuplicateGeometryChecker` | `deduplicateGeometry` | Exact match |
| Fuzzy duplicate geometry | `FuzzyDuplicateGeometryChecker` | `deduplicateGeometry` | `fuzzy: true, tolerance: 0.001` |
| Coinciding geometry | `CoincidingGeometryChecker` | pythonScript: `remove_coinciding_meshes.py` | tolerance=0.001 |
| High vertex count meshes | `HighVertexCountChecker` | `decimateMeshes` | Error-based or ratio-based reduction |
| Duplicate materials | `DuplicateMaterialsChecker` | `optimizeMaterials` | `optimizeMaterialsMode: 2` |

## SimReady Compliance

A technically valid USD file is not necessarily useful for simulation. A mesh with a million triangles is valid USD, but it's a performance nightmare in real-time physics. SimReady adds the rules that define what's *good* for simulation, not just what's technically possible.

**Three-layer architecture:**
- **OpenUSD** — the open-source technology (Pixar) providing the file format and composition engine
- **AOUSD** — the Alliance for OpenUSD, creating formal standardization rules
- **SimReady** — the content specification layer that defines how to structure data for geometry, materials, and physics so tools can understand, validate, and use it

**Profile-based compliance:** Assets don't conform to "all of SimReady" — that's not a thing. Instead, an asset conforms to a specific **profile**: a named, versioned bundle of features for a particular job (e.g., physics-ready prop, articulated robot, placeable visual). Profiles can be neutral (any runtime) or runtime-specific (e.g., Isaac Sim).

**Spec model:** **Requirements** (atomic checkable rules) → **Capabilities** (grouped requirements) → **Features** (runtime-specific bundles) → **Profiles** (complete workflows).

### Stage Requirements

| Requirement | Detection Rule | Fix | Details |
|---|---|---|---|
| Z-up orientation | `UpAxisZChecker` | `editStageMetrics` with `upAxis: 2` | SimReady standard |
| Meters as base unit | `MetersPerUnit1Checker` | `editStageMetrics` with `metersPerUnit: 1.0` | Must equal 1.0 |
| +X forward direction | Visual inspection | pythonScript: `set_forward.py` | `set_x_forward()` |

### Hierarchy Requirements

| Requirement | Detection Rule | Fix | Details |
|---|---|---|---|
| Single root prim | `HierarchyHasRootChecker` | pythonScript: `group.py` | Exactly 1 active root |
| Root is xformable | `RootPrimXformableChecker` | Restructure | Must be strictly Xformable (not Scope) |
| Root at origin | `AssetOriginPositioningChecker` | pythonScript: `transform_stage.py` | No rotation, unit scale, position (0,0,0) |
| Contains mesh geometry | `ContainsMeshChecker` | N/A | At least one mesh required |

### Feature Validation

```bash
# Validate against a specific SimReady feature
aif-pipeline validate input/ output/ --feature minimal_placeable_visual
```

### Optional SimReady Capabilities

These rules validate domain-specific capabilities (not required for all assets):

| Rule | Domain | Description |
|------|--------|-------------|
| `VehicleCapabilityChecker` | Vehicles | Axis-aligned at origin, parts annotated, root untransformed |
| `PedestrianCapabilityChecker` | Characters | Axis-aligned skeleton at origin with skeletal binding |
| `GroundTruthCapabilityChecker` | ML/SDG | Prims must have wikidata_qcode semantic labels |
| `EnvironmentCapabilityChecker` | Environments | Valid OpenDRIVE file; traffic signal labels |
| `EnvironmentLightCapabilityChecker` | Lighting | UsdLux prim or emissive SimPBR material |
| `TrafficLightCapabilityChecker` | Traffic | Directional box with valid signal children |
| `VisualSensorCapabilityChecker` | Sensors | Xformable root, glass as separate prims, material bindings |
| `NonVisualSensorCapabilityChecker` | Sensors | Non-visual material attributes |

## Physics Validation (Optional)

| Rule | Description |
|------|-------------|
| `ArticulationChecker` | Articulation roots cannot be nested; not allowed on static bodies |
| `ColliderChecker` | Collision shape scale must be uniform for Sphere, Capsule, Cylinder, Cone, Points |
| `MassChecker` | Rigid bodies or collision shapes should have mass/inertia specified |
| `PhysicsJointChecker` | Joint Body0/Body1 targets must exist, max one target each |
| `RigidBodyChecker` | Rigid bodies cannot be instanced; must be Xformable without skew |
| `PhysxArticulationChecker` | PhysX articulation validation |
| `PhysxRigidBodyChecker` | PhysX rigid body validation |

## Library Scripts Quick Reference (`so/generic/lib/`)

| Script | Entry Point | Purpose |
|--------|------------|---------|
| `add_layers.py` | `add_layers_from_folder()` | Auto-discovers `./layers/` subfolder, composes as sublayers, updates version metadata |
| `deduplicate_hierarchies_by_display_name.py` | `process_duplicate_hierarchies(use_payloads=False, merge_prototype_hierarchies=False, debug_output=None)` | Groups by display name, converts duplicates to instanceable references or payloads |
| `group.py` | `group_prims(group_name, paths)` | Reparents prims under new Xform using `Sdf.BatchNamespaceEdit`, preserves world-space transforms |
| `material_replacement.py` | `replace_materials()` | Replaces CAD materials with UsdPreviewSurface. Built-in: slate_gray, plastic, shiny_plastic, logo_white, glass, steel, screen |
| `remove_coinciding_meshes.py` | `remove_coinciding_meshes(debug=False)` | Uses SO `findCoincidingMeshes`, then removes duplicates (tolerance=0.001) |
| `set_forward.py` | `set_x_forward()` | Applies 90° Y-rotation, decomposes matrix into T-R-S ops |
| `split_non_composed_by_geom_subsets.py` | `split_all_meshes()` | Splits non-composed meshes by GeomSubsets. **Must run before dedup** |
| `transform_stage.py` | `transform_stage(translate, rotate, scale, up_axis)` | Convenience: `set_z_up()`, `rotate_x/y/z(degrees)`, `identity_transform()` |
| `validate_fix_material_binding_api.py` | `validate_and_fix_material_binding_apis()` | Removes dangling bindings (4 relationship types), applies missing MaterialBindingAPI schema |

## Operation Ordering

Follow this order (same as `so/generic/generic_preset.json`):

1. **`editStageMetrics`** — Normalize units and orientation first
2. **pythonScript** (`split_non_composed_by_geom_subsets.py`) — Split GeomSubset meshes before dedup
3. **pythonScript** (`deduplicate_hierarchies_by_display_name.py`) — Dedup hierarchy branches
4. **`meshCleanup`** — Fix topology before decimation
5. **`decimateMeshes`** — Reduce polygon count
6. **`generateNormals`** — Regenerate after geometry changes
7. **`optimizeMaterials`** (mode 2) — Deduplicate materials
8. **`deduplicateGeometry`** — Instance identical meshes
9. **`pruneLeaves`** — Remove empty nodes from dedup
10. **`optimizePrimvars`** — Clean up unnecessary data
11. **`optimizeMaterials`** (mode 0) — Final material consolidation
12. **pythonScript** (`validate_fix_material_binding_api.py`) — Fix dangling bindings
13. **`removeSmallGeometry`** — Remove degenerate geometry
14. **`pruneLeaves`** — Second pass cleanup
15. **`computeExtents`** — Always last

**Key ordering rules:**
- Stage metrics MUST be first
- GeomSubset split MUST precede dedup
- Cleanup MUST precede decimation
- Normals MUST follow decimation
- Extents MUST be last
- Prune after dedup and after small geometry removal (two passes)
- Material dedup before geometry dedup
- MaterialBindingAPI fix after material operations

## Running Validation

```bash
# OAV standalone (no Kit/GPU required) - AIF category
uv run --directory oav validate --category AIF /path/to/asset.usd
uv run --directory oav validate --rule AIFMetadataChecker /path/to/asset.usd

# Kit-based validation (full rule set including performance rules)
aif-pipeline validate input/ output/
aif-pipeline validate input/ output/ --feature minimal_placeable_visual
aif-pipeline validate input/ output/ --fine-grained  # per-rule timing
```

OAV runs a subset of rules standalone (no GPU). Kit validation runs the full catalog including performance rules (`Usd:Performance`, `Omni:Geometry`) with optional Scene Optimizer C-accelerated implementations.

## Quick Diagnosis Flow

1. **Run validation** (OAV or Kit-based) → identifies specific rule failures
2. **Map each failure** to the symptom→fix tables above
3. **Order the fixes** per the operation ordering section
4. **Compose a preset** or run operations individually

See `.cursor/rules/scene-optimizer-presets.mdc` for how to compose fixes into a preset JSON.

---
> Source: [NVIDIA-Omniverse/aif-pipeline-samples](https://github.com/NVIDIA-Omniverse/aif-pipeline-samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
