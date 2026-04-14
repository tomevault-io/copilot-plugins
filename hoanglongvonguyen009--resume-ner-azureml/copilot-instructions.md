## resume-ner-azureml

> Keep notebooks as orchestration; move reusable logic into src/.


## Notebook Rules

- **Keep notebooks thin**: Notebooks should be orchestration, visualization, and calling functions.
- **Extract reusable logic**: If logic grows (selection, preprocessing, metrics, config parsing), extract it into `src/` as typed functions and import them.
- **Type-check the real logic**: New reusable logic must go into `src/` with type hints so Mypy can check it.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hoanglongvonguyen009) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
