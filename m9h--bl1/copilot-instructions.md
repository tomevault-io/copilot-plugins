## bl1

> BL-1 is a JAX-based in-silico cortical culture simulator. It models dissociated cortical neurons on multi-electrode arrays (MEAs) with biologically detailed spiking neurons, conductance-based synapses, four timescales of plasticity, and closed-loop game experiments. The entire simulation loop is JIT-compiled via `jax.lax.scan` and differentiable through surrogate gradients.

# BL-1 Development Guide

## Project Overview

BL-1 is a JAX-based in-silico cortical culture simulator. It models dissociated cortical neurons on multi-electrode arrays (MEAs) with biologically detailed spiking neurons, conductance-based synapses, four timescales of plasticity, and closed-loop game experiments. The entire simulation loop is JIT-compiled via `jax.lax.scan` and differentiable through surrogate gradients.

## Environment

- **Platform**: DGX Spark (aarch64), NVIDIA GB10 GPU, CUDA 13, JAX 0.9.2
- **Python**: 3.12, venv at `.venv/`
- **NAS**: TrueNAS at `/data` (NFS mount, 41 TB, shared with fedora-legion)
- **Install**: `pip install -e ".[dev]"` then `pip install trackio pynwb dandi optax`

## Commands

```bash
# Tests (must pass before any commit)
make test                              # 536 tests, ~4 min
.venv/bin/pytest tests/test_validation.py -v  # validation framework

# Full validation suite
bash scripts/run_validation.sh --quick  # tests + benchmarks + bio-validation

# Training
python scripts/train_culture.py --from-recording FILE.nwb --n-neurons 5000 --n-epochs 100
python scripts/train_all_sharf.py      # batch training, all 33 recordings

# Dataset analysis
python scripts/analyze_all_datasets.py  # stats for all downloaded data
python scripts/validate_real_data.py    # real vs simulated comparison
```

## Architecture

```
src/bl1/
  core/          # Izhikevich/AdEx neurons, AMPA/NMDA/GABA synapses, integrator
  plasticity/    # STP, STDP, homeostatic, structural
  network/       # Topology, connectivity, Culture factory
  mea/           # Virtual MEA (64-ch, HD-MEA 26K electrodes)
  training/      # Differentiable training loop (trainer.py, loss.py)
  validation/    # Dataset catalog, loaders (NWB/HDF5), comparison framework
  analysis/      # Bursts, criticality, connectivity, information theory
  visualization/ # Raster plots, rates, MEA heatmaps
  games/         # Pong, Doom closed-loop environments
```

## Data on NAS (`/data`)

All large files live on the NAS, visible from both DGX Spark and fedora-legion:

```
/data/datasets/bl1/
  dandi_001611_rat_cortical/   # 2,700 NWB files, 20 GB — rat cortical HD-MEA
  zenodo_sharf_2022/           # 33 HDF5 files, 67 GB — human brain organoid
  osf_dishbrain/               # DishBrain spike data
  results/
    sharf_2022/                # Training results (organized by condition)
      baseline/                #   10 recordings
      development/             #   4 recordings
      drug_dose_response/      #   19 recordings
      trackio/                 #   Experiment tracking logs
      summary_*.csv            #   Spreadsheet-ready results
    dataset_analysis/          # Cross-dataset statistics (JSON)
```

## Validated Parameters (Wagenaar 2006)

These are the calibrated simulation parameters that pass 6/6 bio-validation metrics. Do not change without re-running validation:

- `n_neurons=5000`, `p_max=0.21`, `g_exc=0.12`, `g_inh=0.36`
- AMPA/NMDA split: `nmda_ratio=0.37`
- STP: `U_exc=0.30`, `tau_rec=800ms`
- Burst detection: `threshold_std=1.5`
- Duration: 60s for robust IBI statistics

Config file: `configs/wagenaar_calibrated.yaml`

## Training Pipeline

### How it works

1. Load real recording (NWB or Maxwell HDF5) via `bl1.validation.loaders`
2. Extract targets: firing rate and burst rate from the activity window
3. Build network with `build_connectivity` (sparse BCOO), convert to dense
4. Scale weights down by `init_weight_scale=0.1` (training runs without STP)
5. Forward pass: `simulate(..., surrogate=True)` through `jax.lax.scan`
6. Loss: log-scale FR + differentiable burst proxy + synchrony + weight reg
7. Backward pass: `jax.grad` with SuperSpike surrogate (beta=5.0)
8. Update via Adam + gradient clipping + NaN protection + weight clamping
9. Log to trackio every epoch

### Key files

| File | Purpose |
|------|---------|
| `src/bl1/training/trainer.py` | Core training loop, `train_weights()` |
| `src/bl1/training/loss.py` | Loss function components |
| `src/bl1/training/sharding.py` | Multi-GPU neuron-parallel sharding primitives |
| `scripts/train_culture.py` | CLI entry point, `--from-recording` |
| `scripts/train_all_sharf.py` | Batch training with trackio |
| `configs/wagenaar_calibrated.yaml` | Validated simulation parameters |
| `configs/wagenaar_burst.yaml` | Burst-rate training contract |

### Current state of the training-side knobs

- **FR floor (solved).** Auto-noise calibration (`TrainingConfig.auto_noise=True`)
  scales `I_noise_amplitude` and `init_weight_scale` from the target firing rate
  in two regimes (low <1 Hz: `target*5+0.3`, ws=0.05; high ≥1 Hz: `target*1+0.3`,
  ws=0.50).  Final convergence: 93% of target at 0.3 Hz.

- **Burst-rate matching.** `TrainingConfig` already defaults to
  `sim_duration_ms=5000`, `w_burst_rate=0.5`, `use_stp=True`,
  `target_burst_rate_per_min=8.0` (Wagenaar).  The contract lives in
  `configs/wagenaar_burst.yaml`.  Smoke-tested locally
  (`tests/test_burst_training.py`, marked `@slow`); production validation runs
  via `scripts/slurm_train_burst.sh` on DGX (5K neurons, 100 epochs).

- **Multi-GPU sharding.** Opt-in scaffold for >50K neurons (see GPU Performance
  table below).  Build a mesh with `bl1.training.sharding.make_neuron_mesh()`
  and pass it via `TrainingConfig(mesh=mesh)`; the trainer shards W_exc /
  W_inh / I_external / NeuronState along the neuron axis and lets JAX's
  auto-SPMD partition the scan-body matmuls.  Locally exercised on a fake
  4-device CPU mesh (`tests/test_sharding.py`); real GPU validation pending.

### Success Criteria

| Metric | Threshold | How to check |
|--------|-----------|-------------|
| FR ratio (sim/target) | > 70% | `grep "Final firing" slurm_logs/*.out` |
| Burst ratio (sim/target) | > 60% | `grep "burst rate" slurm_logs/*.out` |
| Bio-validation | 6/6 Wagenaar | `bash scripts/run_validation.sh --quick` |
| Tests | all pass | `make test` (excludes `@slow`) |
| No NaN | 0 NaN epochs | Check training log for `[NaN-protected]` |

### Guardrails

- `make test` must pass before any commit
- Bio-validation must remain 6/6 on Wagenaar metrics
- Never modify `configs/wagenaar_calibrated.yaml` or `configs/wagenaar_burst.yaml`
  without re-running full validation
- Results go to `/data/datasets/bl1/results/` (NAS), not local `results/`
- Use trackio for all training runs
- Submit batch jobs via Slurm (`sbatch`), not serial Python

## GPU Performance

| Neurons | Realtime Factor | Notes |
|---------|----------------|-------|
| 1,000 | 15.6x | |
| 5,000 | 8.6x | Validated config |
| 10,000 | 6.2x | |
| 20,000 | 2.3x | |
| 50,000 | 0.24x | Below realtime |
| 100,000 | 0.06x | Needs multi-GPU |

BCOO/cuSPARSE is the fastest sparse matmul at all scales tested. Event-driven CSC kernels (in `pallas_ops.py`) are correct but slower due to JAX dispatch overhead. Path to >100K is multi-GPU sharding, not custom kernels.

## NSG (Supercomputer) Submission

For large-scale jobs on SDSC Expanse GPUs:

```bash
export NSG_USERNAME=... NSG_PASSWORD=... NSG_APPKEY=...
python scripts/nsg_submit.py --list-tools      # available tools
python scripts/nsg_submit.py --submit           # submit job
python scripts/nsg_submit.py --status JOB_ID    # check status
python scripts/nsg_submit.py --download JOB_ID  # get results
```

Tool: `GPU_PY_EXPANSE` (Python on Expanse GPUs, V100s)

## Slurm (Local DGX Spark)

```bash
# Submit per-recording training array (33 jobs)
sbatch --array=0-32 scripts/slurm_train_sharf.sh

# Submit pooled condition training
sbatch --job-name=bl1-pool-baseline --partition=gpu --gres=gpu:1 --mem=32G --time=02:00:00 \
  --wrap=".venv/bin/python scripts/train_pooled.py --condition baseline"

# Monitor
squeue -o "%.8i %.20j %.4t %.10M %R"
```

## Related Work & Collaboration Opportunities

### DANDI AI Notebooks (Magland et al. 2025)

Paper: "Facilitating analysis of open neurophysiology data on the DANDI Archive
using large language model tools" (bioRxiv 2025.07.17.663965v3)

They built LLM-powered tools for automated DANDI dataset exploration and notebook
generation. GPT-4.1 for chat exploration, Claude Sonnet 4 for notebook generation.
Cost: ~$1.15/notebook. Tested on 12 datasets with expert review.

**Code:**
- Notebook generator: https://github.com/dandi-ai-notebooks/dandi-ai-notebooks-study
- Dandiset Explorer: https://github.com/dandi-ai-notebooks/dandiset-explorer
- Generated notebooks: https://zenodo.org/records/16033603

**Gaps where BL-1 can contribute (see below).**

### Virtual Brain Projects (TVB)

- **tvboptim** (https://github.com/virtual-twin/tvboptim): JAX brain network simulation
  with gradient-based optimization. Wong-Wang, Jansen-Rit, Epileptor models. Uses optax.
  Relevant: `Parameter()` marking system, BOLD monitor, diffrax integration.
- **vbjax** (https://github.com/ins-amu/vbjax): Lean JAX toolkit for virtual brain
  modeling. Euler/Heun/RK4 integrators, custom_vjp sparse matmul, delay helpers,
  BOLD/EEG monitors. Relevant: differentiable sparse ops, Heun integration.

### Beggs Lab (Indiana University)

John Beggs — discoverer of neuronal avalanches (Beggs & Plenz 2003). Book: "The Cortex
and the Critical Point" (MIT Press 2022, open access). BL-1 validates against his
published criticality metrics (branching ratio, -3/2 exponent). The criticality sweep
notebook (notebooks/03_criticality_sweep.ipynb) demonstrates his theory in BL-1.

## Contribution Gaps: DANDI AI Notebooks x BL-1

The Magland et al. pipeline has specific gaps that BL-1 addresses:

### 1. No simulation comparison (biggest gap)
Their notebooks show real data but never compare to a model. BL-1 can generate
a "simulated counterpart" for any DANDI cortical culture recording — run a matched
simulation with extracted targets and produce side-by-side rasters/statistics.
This transforms their descriptive notebooks into model-validation notebooks.

### 2. No MEA-specific analysis
Their tool is generic across all NWB datasets. For cortical culture MEA data
specifically, BL-1 has specialized analysis: burst detection (Wagenaar method),
criticality metrics, STP dynamics, E/I balance estimation. These could be
contributed as "domain plugins" for their notebook generator.

### 3. Spike time format issues unhandled
We discovered that DANDI 001611 stores spike times as sample indices (not seconds)
and Sharf 2022 uses compound HDF5 datasets. Their pipeline likely hits the same
issues. Our activity-window-aware loading and format auto-detection could be
contributed upstream to pynwb or to their inspection tools.

### 4. No differentiable fitting
Their notebooks are read-only analysis. BL-1's training pipeline could be
integrated: after exploring a dataset, the agent generates a training script
that fits BL-1 weights to match the recording's statistics. This is the
"analysis → model → prediction" loop that their system lacks.

### 5. Cost/scale mismatch
Their pipeline costs $1.15/notebook but relies on cloud LLMs. BL-1's analysis
runs locally on GPU at zero marginal cost. Combining their LLM exploration with
BL-1's local GPU analysis could reduce per-dataset cost while adding simulation.

### Concrete contribution plan
1. Submit BL-1's MEA analysis functions as a PR to their notebook generator
2. Add a "simulation comparison" template that their agent can use for
   cortical culture datasets
3. Upstream our NWB loading fixes (sample-index detection, compound datasets)
   to pynwb or their get_nwbfile_info tool
4. Propose a joint notebook: "From DANDI recording to fitted BL-1 simulation"

---
> Source: [m9h/bl1](https://github.com/m9h/bl1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
