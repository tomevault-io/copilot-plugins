## open-normative-eeg

> - `open_normative/` — Core library (parameters, pipeline, preprocessing, spectral, connectivity, channels, normative, compare, source, datasets/)

# Open Normative EEG — Claude Code Guide

## Architecture

- `open_normative/` — Core library (parameters, pipeline, preprocessing, spectral, connectivity, channels, normative, compare, source, datasets/)
- `open_normative/data/` — Pre-computed source localization assets (TMs, forward models, ROI labels, BA labels, DK labels, Brodmann table)
- `scripts/` — CLI tools (build_norms, distribute, downloaders, QC scripts, validation, visualization, cloud_recompute)
- `scripts/normative/` — Dataset-specific QC scripts (dortmund_qc, srm_qc, trt_qc, depress_qc)
- `tests/` — pytest tests
- `Dockerfile` + `requirements-pinned.txt` + `scripts/batch_entrypoint.sh` — container image for AWS Batch (published to `ghcr.io/peak-mind-llc/open-normative-eeg` on main pushes)
- `infra/aws/` — Terraform module: S3 bucket, Batch compute env, queue, IAM, CloudWatch, Budgets
- `.github/workflows/` — `publish-image.yml` (GHCR build on main), `tests.yml` (pytest on PRs)

## Testing

```bash
# Run passing tests (skip asrpy-dependent ones that fail due to upstream numpy 2.x bug)
python -m pytest tests/ --ignore=tests/test_pipeline.py --ignore=tests/test_preprocessing.py
```

## Key Conventions

- Supports both 19-channel 10-20 and 37-channel 10-10 montages (`--channels 19|37`)
- 37ch set: 19 standard + AF3 AF4 FC3 FC1 FC2 FC4 FT7 FT8 CP3 CP1 CP2 CP4 TP7 TP8 PO3 PO4 P1 P2 (matched to pre-computed forward model)
- Dual z-scores: uncorrected (raw band power) + corrected (periodic-only via specparam)
- Distribution honesty (Wood et al. 2024): norm cells carry raw `skewness`/`kurtosis`, a scoring-space `normality_p`, and a `transform_normalized` flag; `compare.py` reports a percentile-derived `robust_z` alongside the parametric z and flags non-normal cells (`parametric_z_unreliable`) and divergence (`z_discrepancy_flag`). See `scripts/distribution_report.py` (disclosure report) and `scripts/compute_trt_reliability.py` (ICC/MDC/heteroscedasticity from the TRT test-retest sessions).
- Log-space subtraction for specparam: `periodic_log10 = log10(full_psd) - log10(aperiodic)`
- `PIPELINE_PARAMS` in parameters.py is the single canonical config dict
- Scripts use argparse, pathlib, checkpoint/resume pattern, logging to stderr
- Workers call `np.seterr(all="warn")` to prevent FloatingPointError from crashing the pool
- ASR includes NaN/Inf detection — reverts to pre-ASR data if ASR corrupts the signal
- ICA catches FloatingPointError/LinAlgError and skips gracefully

## Datasets

| Dataset | Loader | Downloader | QC | Subjects | Ages | Channels |
|---------|--------|------------|-----|----------|------|----------|
| LEMON | `lemon.py` | `lemon_download.py` | `lemon_qc.py` | ~205 | 20-77 | 62ch BrainVision |
| Dortmund | `dortmund.py` | `dortmund_download.py` | `dortmund_qc.py` | ~608 | 20-70 | 64ch BrainProducts |
| SRM | `srm.py` | `srm_download.py` | `srm_qc.py` | 111 | 17-71 | 64ch BioSemi EDF |
| TRT | `trt.py` | `trt_download.py` | `trt_qc.py` | 60 | 18-28 | 64ch BrainVision |
| Depress | `depress.py` | `depress_download.py` | `depress_qc.py` | ~55-70 healthy | 18-24 | 64ch Neuroscan .set |
| HBN | `hbn.py` | `hbn_download.py` | — | ~2800 | 5-21 | 128ch EGI |
| MIPDB | `mipdb.py` | — | — | — | — | — |

All loaders registered in `open_normative/datasets/__init__.py` DATASETS dict.

## Source Localization

### sLORETA Source Power
- Pre-computed transformation matrices map channel power → 2394 MNI voxels → Brodmann areas
- `compute_sloreta_source_power()` accepts `power_key` param: "absolute" or "corrected_absolute"
- Produces both `source_power` and `corrected_source_power` (specparam periodic-only at scalp level, then projected)
- Output channels: `_src_ba_{BA_label}` (e.g., `_src_ba_Brodmann area 17`)

### DICS Beamformer Connectivity (Unified DK-as-Canonical)
- Pre-computed forward model → CSD → DICS beamformer → ROI/DK time courses → spectral connectivity
- **2 direct extractions, 3 atlases:**
  - **18 merged functional ROIs** — direct extraction (`_src_conn_{ROI_A}_{ROI_B}`, 153 pairs). Used because the merged functional ROIs (DLPFC, mPFC, etc.) span multiple DK parcels and are conceptually different from anatomical parcels.
  - **68 individual DK parcels** — direct extraction (`_src_dk_{parcel-lh}_{parcel-rh}`, 2,278 pairs). Canonical anatomical atlas.
  - **~44 Brodmann areas** — DERIVED from DK by aggregation (`_src_ba_conn_BA{A}-{hemi}_BA{B}-{hemi}`). For each BA, average corrected power and connectivity from its mapped DK parcels using the `_DK_TO_BA` table. Single source of truth.
- Connectivity methods: dwPLI, coherence, imaginary coherence
- Network-level aggregation: within/between for 7 networks (from ROI labels)
- Adaptive regularization: tries reg=0.05 → 0.1 → 0.2 → 0.5 on ill-conditioned CSD matrices
- Volume conduction detection: flags high-coherence + low-dwPLI pairs

### DICS Source-Level Specparam (corrected_dics_power)
- Broadband DICS (1-50 Hz) → source time courses → Welch PSD per label → specparam → periodic-only band power
- Theoretically superior to scalp-level correction because aperiodic subtraction happens at the source
- `_broadband_dics()` computes stcs once, shared across ROI and DK label sets
- `_specparam_from_stcs()` extracts time courses + runs specparam per label set
- Produces `corrected_dics_power` and `source_aperiodic_exponent` per label per band
- **Always computed for both ROI (18) and DK (68) when `--source` mode is on with any source connectivity flag.** The `--dk-corrected-power` flag is now a no-op (deprecated).
- **BA derivation:** BA `corrected_dics_power` and BA-BA connectivity are derived from DK in `source_result_to_metrics()`. BA values are weighted averages of DK parcels per the `_DK_TO_BA` table. Each derived value is tagged with `corrected_dics_power_source: "dk_derived"`.

### DK→BA Mapping (`_DK_TO_BA` in `source.py`)
- Standard atlas correspondence between DK parcels and Brodmann areas
- Multiple BAs per parcel (e.g., postcentral → BA1, BA2, BA3): each BA gets weight 1/n
- Inverse `_BA_TO_DK` is built at module load time
- Some DK parcels (entorhinal, parahippocampal, etc.) map to deep BAs that EEG can't see well — included for completeness but values may be noisy

## Lessons Learned

- **Source-level specparam is expensive** — each label requires a Welch PSD + SpectralModel.fit() (~50-200ms per label). For 68 DK parcels, that's 4-15s per subject just for the corrected power step. Made it opt-in via `--dk-corrected-power` flag.
- **Share broadband DICS across label sets** — `_broadband_dics()` runs once and the source estimates are reused for ROI, BA, and DK extraction. Avoids 3x duplicate beamformer compute.
- **ASR can produce NaN/Inf** on ill-conditioned subjects — `apply_asr()` now detects this and reverts to pre-ASR data so ICA doesn't crash on garbage input.
- **np.seterr(all="warn") in workers** prevents FloatingPointError from killing the ProcessPoolExecutor pool when MNE internals trip on overflow (especially `slogdet` in DICS make_filters).
- **Adaptive DICS regularization** — try `reg=0.05 → 0.1 → 0.2 → 0.5` to handle ill-conditioned CSD matrices instead of failing.
- **Cap source DICS epochs** — DICS time-course memory scales linearly with epoch count. The scalp path (`epoch_continuous`) was always capped at `max_epochs=120`; the source path was NOT. When 50% overlap was added it doubled the uncapped source epoch count and OOM-killed Batch workers (~18 GB/subject; SIGKILL/`exit=-9`). `_source_events()` now caps at `max_epochs` (120), keeping the source step ~5 GB/subject regardless of recording length. Profiling proved the CSD fork itself adds ~0 to the peak — the peak is entirely the source step.
- **Specparam at source vs scalp gives different values** — they're not directly comparable. The visualization app must use the same correction method as the norms it's compared against.

### Pre-computed Assets (`open_normative/data/source/`)
- `transformation_matrix_{19,37}ch.npy` — sLORETA TMs
- `forward_{19,37}ch.fif` — forward models
- `src_{19,37}ch.fif` — source spaces
- `roi_labels_{19,37}ch.pkl` — 18 merged DK ROI labels
- `ba_labels_{19,37}ch.pkl` — Brodmann area surface labels (built by `scripts/build_ba_labels.py`)
- `dk_labels_{19,37}ch.pkl` — 68 individual DK parcel labels (built by `scripts/build_dk_labels.py`)
- `LORETA-Talairach-BAs.csv` — voxel-to-BA mapping table

## Build Norms Workflow

```bash
# Generate label pickles (one-time)
python scripts/build_ba_labels.py
python scripts/build_dk_labels.py

# Process a dataset
python scripts/build_norms.py ~/Data/EEG/LEMON/EEG_Raw_BIDS_ID \
    --dataset lemon --condition both --channels 37 \
    --source --ba-connectivity --dk-connectivity --save-psd \
    --output norms_output_lemon -j 5

# Merge multiple datasets
python scripts/build_norms.py --merge \
    --merge-dir norms_output_lemon/subjects \
    --merge-dir norms_output_dortmund/subjects \
    --output norms_output_merged
```

## Output Format

Both `build_norms.py` (normal and merge modes) produce:

```
output_dir/
  norms.json              — full JSON (all cells, ~1 GB for large datasets)
  norms.csv               — flat CSV for R/Python/Excel
  subjects.csv            — per-subject metrics
  norms_psd.npz           — normative PSD (channel-level, for spectral overlays)
  npz/                    — split binary format for fast product loading
    metadata.json         — index: categories, age bins, conditions, cell counts
    scalp_power.npz       — 37ch × 11 bands × power metrics (~1.3 MB)
    scalp_connectivity.npz — 666 electrode pairs × 6 bands × 3 methods (~1 MB)
    source_ba_power.npz   — BA source power + corrected (~360 KB)
    source_ba_connectivity.npz — BA-BA connectivity, DK-derived (~7 MB)
    source_roi_connectivity.npz — 18 merged ROI pairs (~1 MB)
    source_dk.npz         — 68 DK parcel power + 2,278 DK-DK pairs (~14 MB)
    source_network.npz    — network-level connectivity (~176 KB)
    graph_metrics.npz     — global efficiency, char path length (~6 KB)
```

### NPZ Format (`format_version: 4`)

Each `.npz` file contains parallel arrays aligned by index:
- `bins`: age bin labels (U20)
- `conditions`: "ec" or "eo" (U10)
- `sex`: sex variant of the cell — `"pooled"`, `"F"`, or `"M"` (U10). Older v2 bundles without this array default to `"pooled"` on read.
- `reference`: reference frame of the cell — `"average"` or `"csd"` (U15). Older bundles without this array default to `"average"` on read. Source categories are always `"average"`.
- `channels`: channel/pair name (U80)
- `bands`: band name (U64 — fits ratio names like `(Delta+Theta)/(Alpha+Beta)`)
- `metrics`: metric name (U40)
- `mean`, `sd`: float64 arrays for z-score computation
- `n`: int32 sample count
- `log_mean`, `log_sd`: float64 (NaN where not log-transformed)
- `log_transformed`: bool array
- `skewness`, `kurtosis`: float64 raw-distribution shape (NaN where n<3) — Wood et al. (2024) disclosure
- `normality_p`: float64 Shapiro p-value of the **scoring space** (log space for log metrics; NaN where n<3)
- `transform_normalized`: float64 tri-state (NaN=unknown, 1.0=True, 0.0=False) — did the transform achieve Gaussianity?
- `percentile_points`: 1D float64 of the 13 percentile points `[0.5,1,2.5,5,10,25,50,75,90,95,97.5,99,99.5]`
- `percentiles`: (n_cells × 13) float64 matrix aligned with `percentile_points` — lets the product derive a robust (percentile-based) z-score

**Total NPZ size: ~30 MB** vs ~1.1 GB JSON (the percentile matrix added ~6 MB over v1).

For product integration, load only the NPZ files needed:
- Basic z-score report: `scalp_power.npz` + `scalp_connectivity.npz` = ~2.3 MB
- Source analysis: add `source_dk.npz` + `source_ba_*.npz` = ~22 MB more

### NPZ I/O (`open_normative/io.py`)
- `write_norms_npz(cells, output_dir)` — splits NormCells by channel prefix into category NPZ files
- Channel categorization via `_CATEGORY_RULES`: `_pair_` → scalp_connectivity, `_src_dk_` → source_dk, `_src_ba_conn_` → source_ba_connectivity, etc.
- `metadata.json` indexes all NPZ files with cell counts, unique channels/bands/metrics, and file sizes; carries a top-level `references` list of all reference frames present in the bundle, and each category entry includes `unique_references`

## Cloud Pipeline (AWS Batch)

- `scripts/cloud_recompute.py` — orchestrator with subcommands: `submit`, `status`, `logs`, `download`, `list`. Reads `aws-config.yaml` (gitignored; per-user). Writes a `_submission.json` manifest to S3 at submit time so status/logs/download can find jobs by `run_id` later.
- `scripts/batch_entrypoint.sh` — container entrypoint. Switches on `MODE=array|merge` env var. Array mode maps `AWS_BATCH_JOB_ARRAY_INDEX` to a `--subject-range` slice and pipes `build_norms.py` with `--checkpoint-sync s3://...`. Merge mode runs `build_norms.py --merge` and uploads `out/`.
- `infra/aws/` — Terraform module: S3 bucket (runs/ + optional mirrors/), Spot capacity-optimized Batch compute env (uses AWS's service-linked role; do NOT attach a custom `service_role`), queue, two job definitions (array + merge), IAM roles, log group, Budgets, optional SNS.
- **Reproducibility**: BLAS threads pinned to 1 (`OMP_NUM_THREADS`, `OPENBLAS_NUM_THREADS`, `MKL_NUM_THREADS`) in the Dockerfile ENV so outputs are bit-identical across machines. Guarded by `tests/test_determinism.py`.
- **Data source handling**: LEMON mirrored to `s3://<bucket>/mirrors/lemon/` (not on AWS). Dortmund/HBN/MIPDB can be streamed directly from AWS Open Data buckets (`openneuro.org`, `fcp-indi`) — job role already grants read access. Keep runs in `us-east-1` where these buckets live to avoid cross-region egress.
- **Batch quirks learned**: AWS Batch `containerOverrides.command` sets CMD, not ENTRYPOINT — the entrypoint script stays in control, use `MODE` env to branch. Array jobs require `size >= 2`. `aws_batch_compute_environment.compute_resources.instance_type` is a list (singular key, plural value) unlike most AWS provider resources.
- **Cost envelope**: ~$0.60 LEMON full recompute, ~$1 Dortmund. ~$1-2/mo idle storage.
- **Full runbook**: `docs/aws-deployment.md`. **Design rationale**: `docs/aws-deployment-assessment.md`. **Adapting for non-normative experiments**: `docs/adapting-for-new-experiments.md`.

## Distributed Processing (legacy, SSH-based)

- `scripts/distribute.py` — orchestrates build_norms.py across multiple machines via SSH
- Config in `distribute.yaml`: machine names, SSH hosts, NFS paths, worker counts
- Commands: `setup` (create venvs), `run` (dispatch jobs), `status` (check progress), `merge` (combine results)
- NFS share: `/Volumes/dev` (Mac) = `/mnt/dev` (Linux), data at `Data/EEG/EEG/`
- Venvs in home dirs (`~/.eeg-normative-env`), not on NFS (speed)
- **Race condition warning**: Do NOT have multiple machines write to the same `subjects/` directory. Use separate `--output` dirs per machine, then merge. Concurrent writes to the same NFS dir can corrupt checkpoint JSONs.

## Parallelism Notes

- `-j/--jobs` flag for local multiprocessing via ProcessPoolExecutor
- Workers wrap all processing in try/except BaseException to prevent FloatingPointError from killing the pool
- Each worker loads raw data independently (no pickle of Raw objects)
- `iter_subject_files()` on loaders provides lightweight file records for parallel dispatch

---
> Source: [peak-mind-llc/open-normative-eeg](https://github.com/peak-mind-llc/open-normative-eeg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
