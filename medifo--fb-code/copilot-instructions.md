## fb-code

> Validates whether Finnish (FI) maintenance outages propagate to NO3 CNEC

# FI -> NO3 Flow-Based Propagation — Claude Code Project

## What this project does
Validates whether Finnish (FI) maintenance outages propagate to NO3 CNEC
parameters (F0, PTDF_FI, RAM, shadow price) in the Nordic day-ahead
flow-based capacity calculation. Uses real JAO publication-tool data and
ENTSO-E outage events. Runs PanelOLS regressions and produces an HTML report.

## File layout (flat — all in project root)
```
dashboard.py       — tkinter 6-tab GUI  ← MAIN ENTRY POINT
propagation.py     — analytical pipeline (no UI code)
synthetic.py       — synthetic JAO + outage data generator for testing
test_pipeline.py   — full test suite (pytest)
data/
  manual_outages.csv   — hand-curated FI outage events (edit this)
  jao_export.csv       — place your JAO CSV here
results/           — generated outputs (gitignored)
```

## How to run
```bash
# Install dependencies (one time)
pip install pandas numpy matplotlib statsmodels linearmodels requests entsoe-py

# Launch the dashboard
python dashboard.py

# Run tests
pytest test_pipeline.py -v
```

## Key domain facts Claude Code should know
- Nordic RAM formula: RAM = Fmax - FRM - fall + fnrao + AMR - FAAC - IVA
- fall = F_allReference (NOT fref/f0 which is the CGMA-NP reference flow)
- NO3 corridor CNECs: "300KLABU-ORKDAL", "300VERDAL-TUNNSJODAL", "300AURA-VAGAMO" etc.
  Some CNECs appear as UUIDs — the NO3 filter uses biddingZoneFrom/To == "NO3"
- ENTSO-E token: hardcoded as ENTSOE_TOKEN constant in propagation.py
- ENTSO-E A77 = production outages; A78 = transmission outages
- A78 returns mostly forced events (BSNTYPE A54); planned line outages need manual CSV
- FRM is structural (yearly calibration); it should NOT move with individual outages (H6 placebo)
- Two-way clustered SE fails on unbalanced panels -> silent fallback to entity-only clustering
- Outage covariates are zero when no outage overlaps JAO window -> verdicts show as n/a

## Known limitations
- ENTSO-E API returns 403 from cloud/server IPs (works from local machine)
- ENTSO-E A77 returns "File is not a zip file" when no FI production outages exist in window
- IVA is zero on NO3 CNECs in short windows; H5 logit needs longer history
- FRM in synthetic data moves with outages (December 2024 regime change is encoded)
- Two-way clustered SE always falls back to entity-only for unbalanced panels

---
> Source: [MediFo/FB_CODE](https://github.com/MediFo/FB_CODE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
