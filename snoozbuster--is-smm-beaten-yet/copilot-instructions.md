## is-smm-beaten-yet

> Data pipeline scripts — outputs consumed by the app


# Scripts and data pipeline

Scripts in `scripts/` are part of the **data pipeline**: they consume the OneDrive dataset (and optionally the Google sheet) and produce static files the webapp expects.

**Do not** change script output paths or JSON shapes without checking consumers:

- [constants/levelData.ts](../../constants/levelData.ts) — `DATA_ROOT_URL`, expected paths
- Composables and components that fetch `uncleared.json`, `clear_summary.json`, `course-data/{levelId}/`, etc.

**Environment**: Mix of Node (ESM) and Python. See script usage (e.g. `-o=`, `-i=`, `-d=`) in each file.

**Do**: Keep output formats and paths in sync with what the app fetches.

**Don't**: Rename or restructure outputs without updating the app and docs.

---
> Source: [snoozbuster/is-smm-beaten-yet](https://github.com/snoozbuster/is-smm-beaten-yet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
