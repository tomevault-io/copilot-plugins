## usd-issues-catalog

> Known USD issue patterns encountered during CAD-to-USD asset processing. Reference when troubleshooting.


# USD Issues Catalog

Known issue patterns encountered during CAD-to-USD asset processing. Each entry is tagged by tier: `[core]` applies to all SimReady assets, `[aif]` is specific to AI Factory digital twin workflows.

This catalog grows over time. When you encounter a new pattern, add it here following the template below.

## Agent Behavior

When troubleshooting an issue:

1. **Check this catalog first** - scan for a matching symptom in the entries below.
2. If it matches, follow the documented Fix and Prevention steps.
3. If the issue is a validation rule failure, cross-reference with `.cursor/rules/usd-universal.mdc` symptom-to-fix tables for the specific rule-to-operation mapping.
4. If the issue is an infrastructure/runtime error (Kit not found, timeout, OOM), see the "Runtime and Infrastructure Errors" section at the bottom of this file.

---

### [core] Dangling material bindings after CAD conversion

**Symptom:** Render errors, materials appear missing, OAV warnings about unresolved material paths.

**Root cause:** CAD converters create material references that can become invalid after hierarchy restructuring or deduplication. Material prims may be removed while bindings on meshes still reference them.

**Detection:** OAV validation, `optimizeMaterials` analysis mode, visual inspection (meshes render with default grey material).

**Fix:** `optimizeMaterials` operation with `optimizeMaterialsMode: 2` (deduplicate), or the `validate_fix_material_binding_api.py` library script to repair the binding API itself.

**Prevention:** Always run `optimizeMaterials` after any hierarchy restructuring. Place material operations after geometry dedup in presets.

---

### [core] Missing normals after CAD conversion

**Symptom:** Faceted rendering, OAV `NormalsExistChecker` failure, visual artifacts on smooth surfaces.

**Root cause:** Many CAD formats (STEP, JT) do not export surface normals. The CAD converter produces meshes without normal attributes.

**Detection:** OAV `NormalsExistChecker`, `AIFNormalsValidChecker`. Visual: surfaces look faceted when they should be smooth.

**Fix:** `generateNormals` operation with `sharpnessAngle: 60.0, replaceExisting: true`. Adjust sharpness angle for sharper (lower value) or smoother (higher value) results.

**Prevention:** Always include `generateNormals` in presets. Place it after `decimateMeshes` since decimation changes geometry.

---

### [core] Duplicate hierarchy branches from CAD assemblies

**Symptom:** Identical subtrees repeated in the prim hierarchy (for example, 48 identical rack units in a server rack). High prim count, slow load times.

**Root cause:** CAD assembly files contain multiple instances of the same component, but the converter expands them into full separate hierarchies rather than using USD instancing.

**Detection:** Visual inspection of hierarchy tree - look for repeated display names. High prim/mesh counts relative to unique geometry.

**Fix:** The `deduplicate_hierarchies_by_display_name.py` library script (from `so/generic/lib/`) identifies and deduplicates matching branches. For mesh-level dedup, use the `deduplicateGeometry` operation with `duplicateMethod: 2, fuzzy: true`.

**Prevention:** Run hierarchy deduplication early in the preset pipeline (before mesh-level operations).

---

### [core] Wrong stage metrics (Y-up or non-meter units)

**Symptom:** Assets appear sideways or at wrong scale. OAV `UpAxisZChecker` or `AIFMetersPerUnitChecker` failure.

**Root cause:** CAD tools commonly use Y-up orientation and millimeter or inch units. The converter may preserve source metrics.

**Detection:** OAV validation. Visual: asset is rotated 90 degrees or appears tiny/enormous.

**Fix:** `editStageMetrics` operation with `upAxis: 2` (Z-up) and `metersPerUnit: 1.0`.

**Prevention:** Always make `editStageMetrics` the first operation in every preset.

---

### [core] Zero or missing extents on boundable prims

**Symptom:** OAV `ExtentsChecker` failure. Bounding box queries return incorrect results, selection and framing in viewport may fail.

**Root cause:** Extents are not automatically recomputed after geometry operations (decimation, cleanup, dedup). Some CAD converters omit them entirely.

**Detection:** OAV `ExtentsChecker`.

**Fix:** `computeExtents` operation with `paths: []` (all prims).

**Prevention:** Always make `computeExtents` the last operation in every preset.

---

### [core] Degenerate geometry surviving conversion

**Symptom:** Visual artifacts, validation warnings about degenerate faces or non-manifold edges. Possible crashes during decimation.

**Root cause:** CAD models often contain construction geometry, zero-area faces, or self-intersecting geometry that persists through conversion.

**Detection:** OAV `ValidateTopologyChecker`. The `meshCleanup` operation reports statistics on what it fixed.

**Fix:** `meshCleanup` with `removeDegenerateFaces: true, contractDegenerateEdges: true, removeIsolatedVertices: true, removeDuplicateFaces: true`. Follow with `removeSmallGeometry` for sub-threshold remnants.

**Prevention:** Always run `meshCleanup` before `decimateMeshes` in presets - decimation on degenerate input can produce worse results.

---

### [core] Coinciding meshes at same location

**Symptom:** Z-fighting (flickering surfaces), doubled polygon count for overlapping areas, visual artifacts.

**Root cause:** CAD assemblies sometimes contain overlapping or duplicate meshes from different parts of the assembly that occupy the same space.

**Detection:** Visual inspection (z-fighting), the `findCoincidingMeshes` analysis operation, or the `remove_coinciding_meshes.py` library script.

**Fix:** Use the `remove_coinciding_meshes.py` script from `so/generic/lib/` as a pythonScript processor.

**Prevention:** Include coinciding mesh removal in presets for assets known to have complex assemblies.

---

### [core] Asset not at origin after restructuring

**Symptom:** OAV `AIFAssetAtOriginChecker` failure. Asset appears offset from origin in composed scenes.

**Root cause:** Hierarchy restructuring or grouping operations can introduce transforms on the root prim. The root should have an identity transform with all geometry positioned relative to it.

**Detection:** OAV `AIFAssetAtOriginChecker`. Visual: asset is offset from world origin.

**Fix:** Use the `transform_stage.py` library script to reset the root transform, or manually clear xformOps on the root prim.

**Prevention:** After any hierarchy restructuring, verify the root prim has an identity transform.

---

### [aif] Missing AIF metadata on processed assets

**Symptom:** OAV `AIFMetadataChecker` failure. Asset lacks `aif:core:*` and `aif:spec:*` properties.

**Root cause:** Metadata is applied as a separate step after geometry optimization. If the metadata workflow is skipped or the property layer is not composed as a sublayer, the asset has no metadata.

**Detection:** OAV `AIFMetadataChecker`. Check for the presence of a `*_Properties.usda` sublayer.

**Fix:**
1. Create template: `aif-pipeline metadata create --type <type> --output metadata.json`
2. Fill in values
3. Apply: `aif-pipeline metadata apply metadata.json --output <Model>_Properties.usda --prim <DefaultPrim>`
4. Add as sublayer (use `add_layers.py` processor or manual composition)

**Prevention:** Include metadata application in the pipeline. Use `aif-pipeline run` with `--metadata-types` to automate.

---

### [aif] Connection points misaligned after optimization

**Symptom:** Connection point prims (planes/disks) no longer align with equipment openings after decimation or hierarchy changes.

**Root cause:** Connection points are authored against the original geometry. If the geometry is restructured, decimated significantly, or the hierarchy is reorganized, connection points may shift or reference invalid paths.

**Detection:** Visual inspection - overlay connection points on optimized geometry. Check that connection point prims still exist in the expected `ConnectionPoints` scope.

**Fix:** Re-author connection points against the optimized geometry. Save as a separate `<Model>_ConnectionPoints.usd` file and re-compose as a sublayer.

**Prevention:** Author connection points after optimization is complete. If using the `add_layers.py` processor to compose sublayers, run it as the last step.

---

## Runtime and Infrastructure Errors

These are not USD content issues but tooling/environment problems.

---

### Kit path not configured or Kit not found

**Symptom:** `Error: Kit path not configured` or `FileNotFoundError` pointing to a Kit executable.

**Root cause:** No Kit configuration exists, or the configured path is stale (Kit was moved/updated).

**Fix:**
1. Run `aif-pipeline config show` to see current config.
2. Run `aif-pipeline config add <name> --from <kit-project-root>` pointing at the directory containing `repo.bat`/`repo.sh` or the NGC extract root.
3. Do NOT pass `_build/` or subdirectories - the CLI resolves those automatically.

**Prevention:** After updating or moving Kit, re-run `aif-pipeline config add`.

---

### Conversion or optimization timeout

**Symptom:** Process killed or output incomplete. Log shows timeout exceeded.

**Root cause:** Complex CAD files can exceed default timeouts (7200s for conversion, 3600s for optimization).

**Fix:** Increase timeout: `--timeout 14400` (4 hours). For very large assemblies, process sub-assemblies separately.

**Prevention:** Set appropriate timeouts in project config (`conversion.timeout`, `optimization.timeout`).

---

### Out of memory during batch processing

**Symptom:** Process crash, system slowdown, or `MemoryError` in logs.

**Root cause:** Too many concurrent Kit processes for available system RAM. Each Kit instance can use 4-8 GB.

**Fix:** Reduce concurrency: `--concurrent 2` (or 1 for very large files). Close other applications.

**Prevention:** Set conservative concurrency in config. Rule of thumb: available RAM / 8 GB = max concurrent.

---

### Partial batch failure (some files succeed, some fail)

**Symptom:** Mixed results - some output files present, others missing or errored.

**Root cause:** Individual file failures (corrupt CAD, unsupported features, timeouts) while others succeed.

**Fix:**
1. Review logs for specific failure reasons.
2. Fix or exclude problematic files.
3. Resume with `--skip-existing` to process only the remaining files.

**Prevention:** Always use `--skip-existing` for large batches so you can resume without reprocessing.

---

### OAV validation ImportError or module not found

**Symptom:** `ModuleNotFoundError` when running `uv run --directory oav validate`.

**Root cause:** Dependencies not installed in the OAV project environment.

**Fix:** Run `uv sync` from the repo root, or `cd oav && uv sync`.

**Prevention:** Run `uv sync` after every `git pull`.

---
> Source: [NVIDIA-Omniverse/aif-pipeline-samples](https://github.com/NVIDIA-Omniverse/aif-pipeline-samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
