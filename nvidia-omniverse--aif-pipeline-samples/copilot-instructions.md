## usd-aif-profile

> AIF (AI Factory) digital twin profile for USD assets. Covers AIF-specific metadata, connection points, equipment templates, payload structure, and optimization strategy.


# AIF Digital Twin Profile

This profile defines AIF-specific conventions that go beyond SimReady core requirements. It builds on the quality rules in `.cursor/rules/usd-universal.mdc` (which cover core quality and SimReady compliance).

**What's here vs universal rules:**
- Universal rules cover geometry quality, material quality, and SimReady compliance (Z-up, meters, single root, origin, etc.)
- This file covers AIF-only concerns: metadata (`aif:core:*`, `aif:spec:*`), connection points, equipment templates, payload structure, and AIF optimization strategy

## Agent Behavior

### Metadata Requests

When a user asks about AIF metadata or equipment properties:

1. **Determine equipment type** by asking what equipment they are working with:
   - CDU (Coolant Distribution Unit) - 81 properties
   - CRAH (Computer Room Air Handler) - 51 properties
   - UPS (Uninterruptible Power Supply) - 51 properties
   - GB300 Rack - 28 properties (pre-filled with NVIDIA values)
2. **Guide through the workflow:** create template, edit JSON, apply to USD, compose as sublayer, validate.
   - CLI commands are in `.cursor/rules/aif-pipeline-cli.mdc` under Metadata.
3. **After applying metadata,** validate with: `uv run --directory oav validate --rule AIFMetadataChecker <asset>`

### Connection Point Requests

When a user asks about connection points:

1. Confirm optimization is complete first - connection points should be authored against final geometry.
2. Walk through the connection point workflow in the section below.
3. Key naming conventions must match: `<vendor>_<type>_<subtype>_<N>` (e.g., `vertiv_liq_supply_1`).
4. Connection point prims must have `Purpose = guide` and be saved as `<AssetName>_ConnectionPoints.usd`.

### AIF Validation Failures

When `AIFMetadataChecker` or other AIF-specific rules fail:

- **"No properties sublayer found"** - Create and compose a `*_Properties.usda` sublayer using the metadata workflow above.
- **"Missing required attributes"** - Check which `aif:core:*` attributes are missing from the required list below, update the metadata JSON, and re-apply.
- **"Equipment-specific validation failed"** - The `aif:core:assetClass` value does not match the `aif:spec:*` properties present. Ensure the spec properties match the template for the declared asset class.
- **`AIFHierarchyHasRootChecker` failed** - Multiple root prims exist; restructure so there is a single root (excluding `/Render`).
- **`AIFRootIsXformableChecker` failed** - Default prim is not an Xform type; change it to `UsdGeom.Xform`.
- **`AIFAssetAtOriginChecker` failed** - Root prim has a non-identity transform; zero out translation/rotation/scale.

## AIF Hierarchy Structure

A complete AIF asset composes these layers:

```
/<DefaultPrim>              (Xform, equipment root)
├── <Geometry>              (Xform or Mesh prims - CAD-converted geometry)
├── ConnectionPoints/       (Scope - thermal, electrical, airflow interfaces)
│   ├── <vendor>_liq_supply_*     (Plane/Disk, Purpose=guide)
│   ├── <vendor>_liq_return_*     (Plane/Disk, Purpose=guide)
│   ├── <vendor>_electrical_*     (Plane/Disk, Purpose=guide)
│   ├── <vendor>_airvent_intake_* (Plane/Disk, Purpose=guide)
│   └── <vendor>_airvent_outflow_*(Plane/Disk, Purpose=guide)
└── [sublayers]
    ├── <Model>_Properties.usda   (AIF metadata layer)
    └── <Model>_ConnectionPoints.usd (connection point layer)
```

## Metadata Properties

AIF metadata uses two namespaces applied as a separate USDA property layer.

### `aif:core:` — Common Properties (all equipment)

Applied to the equipment root prim. Key properties:

| Property | Type | Description |
|---|---|---|
| `aif:core:manufacturer` | string | Equipment manufacturer name |
| `aif:core:modelNumber` | string | Equipment model number |
| `aif:core:overallGeometryDimensions` | float3 | Overall geometry W x D x H (mm) |
| `aif:core:weight` | float | Weight in kilograms |
| `aif:core:height` / `width` / `depth` | float | Individual dimensions in mm |
| `aif:core:assetClass` | string | Class of AI Factory equipment |
| `aif:core:assetVersion` | string | Design revision of digital twin asset |
| `aif:core:assetCreationDate` | string | ISO 8601 date (YYYY-MM-DD) |
| `aif:core:assetDescription` | string | Human-readable description |
| `aif:core:sceneOptimizerVersion` | string | SO version (tool-managed, excluded from validation) |
| `aif:core:assetValidatorVersion` | string | Validator version (tool-managed, excluded from validation) |

All numeric values use SI units (meters, kilograms, Kelvin, watts) unless noted.

### `aif:spec:` — Equipment-Specific Properties

Vary by equipment type. Create templates with:

```bash
aif-pipeline metadata create --type <type> --output <file>.json
```

| Type | Description | Total Properties |
|------|-------------|-----------------|
| `cdu` | Coolant Distribution Unit | 81 (20 common + 61 specific) |
| `crah` | Computer Room Air Handler | 51 (20 common + 31 specific) |
| `ups` | Uninterruptible Power Supply | 51 (20 common + 31 specific) |
| `gb300_rack` | NVIDIA DGX GB300 Rack | 28 (20 common + 8 specific, pre-filled) |

Templates are defined in `metadata/templates/` with source CSVs in `metadata/templates/source/`.

### AIFMetadataChecker Validation Details

The OAV `AIFMetadataChecker` rule validates:

1. **Sublayer check:** Looks for any `.usda` sublayer with "properties" in filename
2. **DefaultPrim check:** Properties layer must have a defaultPrim
3. **Attribute presence:** Prim must have attributes with `aif:` prefix
4. **Required common attributes** (warning if missing):
   - `aif:core:assetClass`, `aif:core:assetDescription`, `aif:core:assetVersion`
   - `aif:core:depth`, `aif:core:height`, `aif:core:width`, `aif:core:weight`
5. **Tool-managed attributes** (excluded from validation): `aif:core:sceneOptimizerVersion`, `aif:core:assetValidatorVersion`
6. **Equipment-specific check:** If `aif:core:assetClass` is set, validates `aif:spec:*` properties against templates

### Metadata Workflow

1. Create template: `aif-pipeline metadata create --type cdu --output cdu.json`
2. Edit JSON to fill in values
3. Generate USDA layer: `aif-pipeline metadata apply cdu.json --output <Model>_Properties.usda --prim <DefaultPrim>`
4. Add as sublayer to main asset (drag into Layers panel or use `add_layers.py` processor)
5. Update when templates change: `aif-pipeline metadata update <Model>_Properties.usda`

Naming convention: `<ModelName>_Properties.usda`

## Connection Points

Geometry prims (planes or disks) positioned at equipment openings for simulation runtime connectivity.

### Naming Conventions

| Type | Naming Pattern | Example |
|------|---------------|---------|
| Liquid supply pipe | `<vendor>_liq_supply_*` | `vertiv_liq_supply_primary_01` |
| Liquid return pipe | `<vendor>_liq_return_*` | `trane_liq_return_secondary` |
| FWS supply pipe | `<vendor>_fws_supply_piping_connection_*` | `vertiv_fws_supply_piping_connection_main` |
| FWS return pipe | `<vendor>_fws_return_piping_connection_*` | `vertiv_fws_return_piping_connection_main` |
| TCS supply pipe | `<vendor>_tcs_supply_piping_connection_*` | `vertiv_tcs_supply_piping_connection_main` |
| TCS return pipe | `<vendor>_tcs_return_piping_connection_*` | `vertiv_tcs_return_piping_connection_main` |
| Electrical socket | `<vendor>_electrical_nominal_voltage_*` | `trane_electrical_nominal_voltage_main` |
| Airflow intake | `<vendor>_airvent_intake_*` | `trane_airvent_intake_frontplate` |
| Airflow outflow | `<vendor>_airvent_outflow_*` | `trane_airvent_outflow_cabinet_top` |

### Connection Point Requirements

- Placed inside a `ConnectionPoints` scope under the default prim
- Geometry matches size and location of actual equipment openings
- Purpose set to `guide` (not rendered visually, no materials needed)
- Saved as separate file: `<AssetName>_ConnectionPoints.usd`
- Composed as sublayer into the main asset

### Connection Point Workflow

1. Open main geometry in USD Composer, note the defaultPrim name
2. Create new empty scene (Z-up, meters)
3. Reference main geometry as payload
4. Create `ConnectionPoints` scope, add Plane/Disk prims at openings
5. Name per conventions, set Purpose to `guide`
6. Delete payload reference, keep only ConnectionPoints
7. Rename defaultPrim to match original
8. Save as `<AssetName>_ConnectionPoints.usd`
9. Sublayer into main asset

## AIF Optimization Strategy

For AIF digital twin assets, optimize aggressively for real-time factory scene performance:

### Instancing

Use heavy instancing for repeated equipment (racks, CDUs on factory floor):
- `deduplicateGeometry` with `fuzzy: true, duplicateMethod: 2`
- `deduplicate_hierarchies_by_display_name.py` for hierarchy-level dedup
- Consider payloads for large assets (`use_payloads=True`)

### Decimation

Aggressive decimation acceptable — visual fidelity matters less than runtime performance:
- `decimateMeshes` with error-based tolerance (`maxMeanError: 0.0001`)
- Pin boundaries to preserve connection surfaces (`pinBoundaries: true`)
- GPU acceleration for large meshes (`gpuVertexCountThreshold: 500000`)

### Materials

- Deduplicate and consolidate (`optimizeMaterials` mode 2 then mode 0)
- Move all materials under `/World/Looks`
- Remove unnecessary primvars after material optimization
- For CAD materials, use `material_replacement.py` to standardize to UsdPreviewSurface

### Payload Structure

Vendor presets in `so/vertiv/` demonstrate internal/external payload splitting:
- Main preset (`<model>.json`) — geometry optimization
- Payloads preset (`payloads.json`) — splits into internal/external layers
- Reference these as patterns for new equipment

## Reference Presets

| Folder | Equipment | Pattern |
|--------|-----------|---------|
| `so/generic/` | Any CAD asset | Universal optimization template |
| `so/spt/gb300/` | GB300 rack | Hierarchy dedup, instancing, alternate strategies |
| `so/trane/` | Trane 220LL chiller | Material replacement, stage prep |
| `so/vertiv/<model>/` | Vertiv equipment | Payload assembly, occluder creation |

## Validation

After processing, validate with AIF rules:

```bash
# Standalone OAV (no Kit required)
uv run --directory oav validate --category AIF /path/to/asset.usd

# Kit-based validation
aif-pipeline validate output/ validation/ --stage post
```

AIF-specific check: `AIFMetadataChecker` (metadata presence, required attributes, equipment-specific properties). All other OAV rules under the AIF category are SimReady core requirements documented in `.cursor/rules/usd-universal.mdc`.

---
> Source: [NVIDIA-Omniverse/aif-pipeline-samples](https://github.com/NVIDIA-Omniverse/aif-pipeline-samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
