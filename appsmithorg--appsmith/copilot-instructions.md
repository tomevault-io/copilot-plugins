## appsmith

> When you edit `values.yaml`, regenerate both derived files before committing:

# Helm Chart

## Regenerating derived files

When you edit `values.yaml`, regenerate both derived files before committing:

```bash
# 1. Regenerate the JSON schema
helm schema \
  --schema-root.title "Appsmith Helm chart values" \
  --schema-root.id "https://helm.appsmith.com/values.schema.json" \
  -o values.schema.json

# 2. Regenerate the README
helm-docs --sort-values-order file
```

The CI workflow `.github/workflows/helm-docs.yml` enforces both are up to date.

## Annotation format

Each documented value in `values.yaml` carries up to three comment layers (all plain YAML comments):

- `## @param key.name Description` — Bitnami readme-generator compatibility
- `# -- Description` — helm-docs picks this up as the value description
- `# @section -- Section Name` — helm-docs groups the value into a named section table

When adding a new value, include at least the `# --` and `# @section --` lines so it lands in the correct README section. The `## @param` line is optional but preferred for consistency.

---
> Source: [appsmithorg/appsmith](https://github.com/appsmithorg/appsmith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
