## mp-shock

> This project combines three monetary-policy shock series into a common monthly-date file.

# MP_Shock project memory

## Purpose

This project combines three monetary-policy shock series into a common monthly-date file.

## Data flow

- `scripts/02_merge_three_shocks.py` reads:
  - `data/raw/brw-shock-series.csv` → `BRW_monthly`
  - `data/raw/monetary-policy-surprises-data.xlsx`, sheet `Monthly (original)` → `MPS`, `MPS_ORTH`
  - `data/raw/identkurto_replicate/work/data.csv` → `MP1`
- It writes `data/clean/final_three_mp_shocks.csv` with columns `date`, `BRW_monthly`, `MPS`, `MPS_ORTH`, and `MP1`.
- Dates are normalized to the first day of each month. MP1 stays event-level, so months with multiple events retain multiple output rows.
- The BRW–MPS merge is `1:1`; the subsequent MP1 merge is `1:m`. Preserve these `validate=` checks unless the source grain changes.

## Coding conventions

- Follow `scripts/SKILL.md` for pandas/numpy scripts. It is registered as the project skill at `.codex/skills/pandas-chained-pipeline-style/SKILL.md`.
- Use Org-style section banners (`# * NAME #`), `pathlib.Path`, and parenthesized pandas method chains.
- Use `.assign()` for derived columns, `.filter()` for final column selection, named aggregations, and explicit merge cardinalities.
- Keep the `if __name__ == "__main__":` block as a flat pipeline driver; do not add a `main()` function or CLI unless requested.

## Validation

- Perform an in-memory load and merge after changing the script. With the current source data, the merged result has 587 rows and the five columns listed above.

---
> Source: [jul225-blip/mp_shock](https://github.com/jul225-blip/mp_shock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
