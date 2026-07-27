## dataportals-registry

> Rules for editing catalog YAML entries in the dataportals-registry


# Catalog entry YAML edits

When adding or editing data catalog YAML files in this repository:

## File location and naming

- **Filename** must equal the catalog `id`: lowercase, no special characters (e.g. `catalogdatagov.yaml` → `id: catalogdatagov`).
- **Path**: `data/entities/COUNTRY_CODE/type/` for verified entries, or `data/scheduled/` for unverified. Use `Federal/` or subregion code (e.g. `US-CA/`) under country as appropriate. Type folders: `opendata/`, `geo/`, `scientific/`, `microdata/`, `indicators/`, `ml/`, `search/`, `api/`, `marketplace/`, `other/`.

## UID and new entries

- **Do not** set or edit `uid` manually. After adding new YAML files, run:
  ```bash
  python scripts/builder.py assign
  ```
- Prefer creating new entries via:
  ```bash
  python scripts/builder.py add-single --url "..." --software ckan --catalog-type "Open data portal" ...
  ```
  when applicable; otherwise create the YAML under the correct path and then run `assign`.

## After editing

- Run schema validation:
  ```bash
  python scripts/builder.py validate-yaml
  ```

## Reference

- **Schema**: [data/schemes/catalog.json](data/schemes/catalog.json)
- **Required fields**: `id`, `uid`, `name`, `link`, `catalog_type`, `access_mode`, `status`, `software`, `owner`, `coverage`
- **Conventions**: [CONTRIBUTING.md](CONTRIBUTING.md), [openspec/project.md](openspec/project.md)

---
> Source: [commondataio/dataportals-registry](https://github.com/commondataio/dataportals-registry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
