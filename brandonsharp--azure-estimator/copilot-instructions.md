## azure-estimator

> Guidance for AI coding agents and contributors working in this repository.

# AGENTS

Guidance for AI coding agents and contributors working in this repository.

## Mission

Maintain and extend an Azure cost estimator that:

- Reads estimate/profile/manual-pricing YAML
- Resolves prices from Azure Retail Prices API
- Emits deterministic monthly cost line items and reports

## Core workflow

1. Inputs are merged with precedence:
   - `src/configs/manual-pricing.yaml`
   - `src/configs/profiles.yaml`
   - estimate YAML file
2. Resource entries are dispatched by `type` to emitter functions:
   - `emit_<type>` in `src/emitters/*.sh`
3. Emitters call `price_for` and `add_item`
4. Outputs are written under `output/...`

## Key files

- `src/az-cost.sh`: merge logic, price query/cache, resource dispatch, reporting
- `src/emitters/baseline.sh`: baseline/shared emitters
- `src/emitters/spokes.sh`: spoke/resource emitters
- `src/emitters/shared.sh`: shared emitter helpers
- `src/configs/service-meter-map.yaml`: service/meter matching rules
- `src/configs/estimate.schema.json`: estimate schema
- `src/configs/profiles.schema.json`: profiles schema
- `doc/adding-resource-types.md`: canonical process for new resource types

## Edit rules for agents

- Prefer minimal, targeted edits; avoid broad refactors unless requested.
- Preserve existing CLI behavior and output columns/field names.
- Keep paths aligned with current `src/` layout.
- When changing resource inputs, update schemas and docs in the same change.
- Do not silently change precedence semantics.
- Avoid adding dependencies unless necessary.

## If adding a new resource type

1. Add or update meter rules in `src/configs/service-meter-map.yaml`.
2. Implement `emit_<type>` in the appropriate emitter file.
3. Ensure dispatch naming works (`-` maps to `_`, alias handling if needed).
4. Add example usage in configs/docs as appropriate.
5. Update `src/configs/estimate.schema.json` resource definitions.
6. Ensure `src/configs/profiles.schema.json` still references the updated resource schema.
7. Run estimator and verify source/quantities/extensions in output.

## Validation checklist

Before finishing changes, agents should validate:

- `jq . src/configs/estimate.schema.json`
- `jq . src/configs/profiles.schema.json`
- VS Code diagnostics are clean for touched files
- Run a representative estimate:

```bash
./src/az-cost.sh \
  estimates/test-estimate.yaml \
  src/configs/profiles.yaml \
  src/configs/service-meter-map.yaml \
  output/test-estimate \
  src/configs/manual-pricing.yaml
```

## Task usage

Preferred task for estimates:

- `Estimate: Run az-cost for active file`

VM SKU discovery task:

- `Azure: List VM SKUs in region`

## Common pitfalls

- Missing quantity defaults can yield `0` extended cost even with API price found.
- Meter filters that are too broad or too narrow produce wrong/missing matches.
- Profile mismatch causes unintended baseline/spoke resources.
- Schema drift leads to poor editor completion and invalid config edits.

## Out of scope unless requested

- Changing report format or output file names
- Reworking merge precedence model
- Large restructuring of shell scripts

---
> Source: [BrandonSharp/azure-estimator](https://github.com/BrandonSharp/azure-estimator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
