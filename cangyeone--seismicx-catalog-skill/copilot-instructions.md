## seismicx-catalog-skill

> This repository is an agent-agnostic skill for the full earthquake detection-to-catalog workflow. Use it when a user asks for earthquake phase detection/picking, multi-station phase association, event location, ML magnitude, activity analysis, mapping, or focal-mechanism workflows from local waveform data.

# SeismicX Catalog Agent Instructions

This repository is an agent-agnostic skill for the full earthquake detection-to-catalog workflow. Use it when a user asks for earthquake phase detection/picking, multi-station phase association, event location, ML magnitude, activity analysis, mapping, or focal-mechanism workflows from local waveform data.

## Canonical Workflow

- Read `SKILL.md` first; it is the source of truth for the workflow.
- Load `references/data-contracts.md` before converting waveform, pick, event, station, magnitude, or mechanism tables.
- Load `references/association-location.md` before REAL, GaMMA, grid location, `bayes_location`, NLLoc, or SeismicX-Location work.
- Load `references/focal-mechanism.md` before first-motion polarity, HASH/pyhash, or focal-mechanism products.
- Load `references/quality-control.md` before final delivery.

## Core Commands

```bash
python scripts/seismicx_catalog.py list-models
python scripts/seismicx_catalog.py catalog -w <waveforms> -s stations.csv -v velocity_model.csv -o work/catalog_run --association-method gamma
python scripts/seismicx_catalog.py associate --method real -p work/picks.csv -s stations.csv -o work/events_real.csv --assignments work/assignments.csv --associated-picks work/picks_associated.csv
python scripts/seismicx_location.py train --stations stations.csv --samples travel_times.csv --projection-origin "$SEISMICX_PROJECTION_ORIGIN" --output work/seismicx_location.pt
python scripts/seismicx_catalog.py locate --method seismicx-location -p work/picks_associated.csv -s stations.csv -o work/events_seismicx.csv --seismicx-location-checkpoint work/seismicx_location.pt --seismicx-location-projection-origin "$SEISMICX_PROJECTION_ORIGIN"
python scripts/seismicx_catalog.py build-tools --tool all --tools-dir external --skip-build -o work/tools_manifest.json
```

The final one-shot output is `work/catalog_run/catalog_final.csv`, produced after waveform scanning, phase detection, association, location, ML, and focal-mechanism steps. For production association, prefer GaMMA or REAL over the simple smoke-test associator; the end-to-end `catalog` command requires `--smoke-test-simple` before it will run the simple path.

Continuous-waveform phase picking must remain unfiltered. Do not apply bandpass, highpass, or lowpass filters before the PNSN picker or inside the default `pick`/`catalog` detection path. Use filtering only when the user explicitly requests a classic STA/LTA smoke-test experiment or during later response/magnitude processing.

ML magnitude calculation should follow the seedtools-style path: remove response to velocity, simulate/integrate to displacement, measure SME/SMN horizontal amplitudes in micrometers, and apply the selected regional R curve such as `R13`. Treat raw-scaled ML as a smoke-test fallback only.

## External Tools

- Install GaMMA for production association: `python -m pip install "git+https://github.com/wayneweiqiang/GaMMA.git"`.
- Download PNSN, REAL, `bayes_location`, and `seismological-ai-tools` with `build-tools --tool all --skip-build`.
- Use the bundled Python REAL backend for homogeneous-velocity association without compilation.
- Compile REAL with `build-tools --tool real` when a C REAL binary or layered travel-time table is required.
- Compile HASH/pyhash only from an explicit local source tree: `build-tools --tool hash --hash-source ./pyhash`.
- Do not commit external cloned repositories, generated catalogs, raw waveform archives, compiled binaries, or large model weights.
- SeismicX-Location is local-only. Never send its station/event data or projection origin to a remote service. The projection origin must be supplied at runtime, is redacted from catalog command logs, and is not persisted in its checkpoint.

## Validation

Before finishing changes, run:

```bash
python -m py_compile scripts/seismicx_catalog.py
python scripts/seismicx_catalog.py list-models
python scripts/smoke_test.py
```

If the skill structure changed, also run the local skill validator when available:

```bash
python ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py .
```

---
> Source: [cangyeone/seismicx-catalog-skill](https://github.com/cangyeone/seismicx-catalog-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
