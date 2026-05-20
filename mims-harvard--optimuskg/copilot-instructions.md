## optimuskg

> When editing a node file, it is required to also edit the corresponding YAML files in the catalog to be in sync.

When editing a node file, it is required to also edit the corresponding YAML files in the catalog to be in sync.

Never delete the checksum property in the catalog. If the code that generates a file has changed, then rerun the node and use the CLI to find the new checksum. Then update the catalog with the new checksum.

When editing a node file, always rerun all downstream nodes. This is done by looking for the catalog id of the outputs of the node being edited using grep. And doing this recursively until the end of the pipeline.

---
> Source: [mims-harvard/OptimusKG](https://github.com/mims-harvard/OptimusKG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
