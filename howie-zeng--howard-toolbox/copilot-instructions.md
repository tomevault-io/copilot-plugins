## memories

> Project-specific domain corrections and facts for howard-toolbox


# Project Memories

Project-specific facts and corrections. Cap ~20 entries. When exceeded, consolidate
into `.cursor/skills/howard-toolbox/SKILL.md` (with diff preview + approval).

## Domain Corrections

- Dial value `1.0` means "no dial" and removes the Shock entry entirely -- it does not set an identity multiplier.
- Tracking files use `_YYYYMMDD.xlsx` suffix in the filename for date identification.
- `pd.ExcelFile` in `dial_utils.py` is not context-managed -- wrap in `with` if modifying that code.
- The `_ensure_asset` function in `emailer/run.py` copies images to `assets/` and rewrites markdown paths; do not duplicate this logic.
- `format_excel.py` loads workbooks twice: `data_only=True` for value sampling, normal mode for editing. Both handles must be used correctly.
- Spec files (`*_all_dials_spec.json`) are per-product configurations. They are generated with `--generate-spec` and applied with `--spec` -- never edit model JSONs directly.
- The `formatter/` templates key on original header names. Column renames are visual-only and applied last.

---
> Source: [howie-zeng/howard-toolbox](https://github.com/howie-zeng/howard-toolbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
