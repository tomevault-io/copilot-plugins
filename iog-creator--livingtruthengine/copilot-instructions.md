## livingtruthengine

> Ensure SSOT metadata index exists, validates, and stays fresh


# Policy
- The file `reports/ssot_meta_index.json` must exist and parse as JSON.
- It must validate (schema + suggested types) and be **fresh** relative to rules/docs.
- `make reports` runs `meta-index` and `meta-validate`.

# How to comply
```bash
make meta-index
make meta-validate
```

### SSOT Meta

```yaml
owner: researcher-ssot
severity: high
tags: [ssot, metadata, validation]
phase: 9.5.7.4.12
dependsOn: [meta-index, meta-validate]
autofixAllowed: true
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iog-creator) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
