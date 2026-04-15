## schumann

> This repository implements advanced EEG signal processing and analysis focused on **Schumann Resonance** detection and brain-field coherence. The project analyzes EEG data to detect "ignition" events (transient neural activations) and their relationship to Earth's Schumann Resonance frequencies (~7.83 Hz fundamental + harmonics).

# CLAUDE.md - Schumann Resonance EEG Analysis

## Project Overview

This repository implements advanced EEG signal processing and analysis focused on **Schumann Resonance** detection and brain-field coherence. The project analyzes EEG data to detect "ignition" events (transient neural activations) and their relationship to Earth's Schumann Resonance frequencies (~7.83 Hz fundamental + harmonics).

### Core Hypothesis
The project investigates whether EEG signals show coherence with Schumann Resonance frequencies during specific conscious states ("ignition" events), suggesting potential brain-field coupling.

## Quick Start

```python
# Standard imports in notebooks
import sys; sys.path.insert(0, './lib')
import utilities
import harmonics
import detect_ignition
# ... other lib modules

# Key constants
FS = 128  # Sampling rate (Hz)
ELECTRODES = ['AF3','AF4','F7','F8','F3','F4','FC5','FC6','P7','P8','T7','T8','O1','O2']
RANGES = {'Delta':[1,4],'Theta':[4,8],'Alpha':[8,12],'BetaL':[12,16],'BetaH':[16,25],'Gamma':[25,45]}

# Load EEG data
RECORDS = utilities.load_eeg_csv("data/file.csv", electrodes=ELECTRODES)

# Estimate Schumann harmonics from data
HARMONICS = harmonics.estimate_sr_harmonics(RECORDS, sr_channel='EEG.F4', fs=128,
    f_can=(7.83, 14.3, 20.8, 27.3, 33.8), search_halfband=0.8)

# Detect ignition windows
out, IGNITION_WINDOWS = detect_ignition.detect_ignitions_session(RECORDS, ...)
# Note: IGNITION_WINDOWS is List[Tuple[int, int]] — integer seconds (rounded)
```

## Directory Structure

```
schumann/
├── lib/                      # Core analysis modules (50+ Python files)
│   ├── utilities.py          # Data loading, filtering, basic analysis
│   ├── harmonics.py          # Schumann spike detection (Morlet wavelet)
│   ├── detect_ignition.py    # Ignition event detection
│   ├── fooof_harmonics.py    # FOOOF/SpecParam-based harmonic detection
│   ├── cross_frequency.py    # PAC, bicoherence, waveform shape
│   ├── criticality.py        # 1/f slope, DFA, avalanches
│   ├── network_geometry.py   # State-space embeddings (UMAP)
│   ├── information_flow.py   # VAR, PDC, DTF, transfer entropy
│   ├── harmonic_coherence.py # Schumann-EEG coherence signatures
│   ├── session_metadata.py   # Metadata parsing for batch processing
│   ├── psd_waterfall.py      # Cross-session PSD aggregation
│   ├── test.py               # Visualization (six_panel, FeaturePackCfg)
│   └── ... (40+ specialized modules)
├── data/                     # EEG CSV files (gitignored)
├── exports*/                 # Analysis outputs (gitignored)
├── Basics.ipynb              # Main analysis notebook
├── Six Panel.ipynb           # Dashboard visualizations
└── Ignition.ipynb            # Ignition-focused analysis
```

## Data Format

### Input: EEG CSV Files
- **Emotiv format**: Columns like `EEG.AF3`, `EEG.F4`, etc.
- **Muse format**: Columns `eeg_1`-`eeg_4` mapped to `EEG.AF7`, `EEG.AF8`, `EEG.TP9`, `EEG.TP10`
- **Timestamp**: Either explicit `Timestamp` column or inferred at 128 Hz

### RECORDS DataFrame Columns (after loading)
```
Timestamp                     # Time in seconds
EEG.<electrode>               # Raw EEG values
EEG.<electrode>.FILTERED      # 1 Hz high-pass filtered
EEG.<electrode>.FILTERED.POW  # Squared power
EEG.<electrode>.<band>        # Band-passed signal (e.g., EEG.F4.Alpha)
POW.EEG.<electrode>.<band>    # Band power
POW.EEG.<electrode>.<band>.REL  # Relative band power
```

## Key Schumann Resonance Frequencies

| Harmonic | Canonical (Hz) | Description |
|----------|---------------|-------------|
| f₀ | 7.83 | Fundamental |
| f₁ | 14.3 | 2nd harmonic |
| f₂ | 20.8 | 3rd harmonic |
| f₃ | 27.3 | 4th harmonic |
| f₄ | 33.8 | 5th harmonic |
| f₅ | 40.3 | 6th harmonic |

Subharmonics: 3.915 (f₀/2), 2.61 (f₀/3), 1.9575 (f₀/4), 1.566 (f₀/5)

### Session-Specific Harmonic Labels
For session-specific fitting, custom harmonic labels are used:
```python
LABELS = ['sr1', 'sr1.5', 'sr2', 'sr2o', 'sr2.5', 'sr3', 'sr4', 'sr5', 'sr6']
CANON = [7.6, 10, 12, 13.75, 15.5, 20, 25, 32, 40]  # Session-specific centers
HALF_BW = [0.6, 0.618, 0.7, 0.75, 0.8, 1, 2, 2.5, 3]  # Per-harmonic bandwidths
```

## Core Analysis Pipeline

### 1. Load & Preprocess
```python
RECORDS = utilities.load_eeg_csv(filename, electrodes=ELECTRODES, device="emotiv", fs=128)
```

### 2. Estimate Session-Specific Harmonics
```python
HARMONICS = harmonics.estimate_sr_harmonics(
    records=RECORDS,
    sr_channel='EEG.F4',           # Single channel or list of channels
    fs=128,
    f_can=(7.83, 14.3, 20.8, 27.3, 33.8),
    search_halfband=0.8,
    nperseg_sec=32.0,              # Spectral resolution
    overlap=0.5,
    time_col="Timestamp",
    comb="median"                  # 'median' | 'mean' | 'sum' for multi-channel
)
```

### 3. Detect Ignition Windows
```python
out, IGNITION_WINDOWS = detect_ignition.detect_ignitions_session(
    RECORDS,
    sr_channel='EEG.F4',
    eeg_channels=['EEG.O1','EEG.O2',...],
    center_hz=HARMONICS[0],
    half_bw_hz=0.5,                # Can be scalar or list (one per harmonic)
    z_thresh=2.5,

    # Harmonic parameters
    harmonics_hz=[7.83, 14.3, 20.8, 27.3, 33.8],  # Explicit frequencies
    labels=['sr1', 'sr2', 'sr3', 'sr4', 'sr5'],   # Custom labels
    harmonic_method='psd',         # 'psd', 'fooof_session', 'fooof_event', 'fooof_hybrid'

    # FOOOF parameters (when harmonic_method uses FOOOF)
    fooof_freq_range=(5, 45),
    fooof_max_n_peaks=15,
    fooof_peak_threshold=2.0,
    fooof_min_peak_height=0.05,
    fooof_peak_width_limits=(1.0, 8.0),
    fooof_match_method='power',    # 'distance', 'power', 'average'

    nperseg_sec=4.0,               # Spectral resolution
)
# Returns: (output_dict, List[Tuple[int, int]])
# Note: Second element contains INTEGER time values (rounded seconds)
```

### 4. Run Specialized Analyses
```python
# Schumann wavelet detection with heatmaps
fused = harmonics.detect_and_plot_schumann_microgrid_with_global_tf(
    RECORDS, signal_col='EEG.F4', f0=7.83, n_harmonics=5, show=True)

# Cross-frequency coupling
res = cross_frequency.run_crossfreq_suite_records(RECORDS, ignition_windows=IGNITION_WINDOWS)

# Criticality analysis
crit = criticality.run_criticality_analysis(RECORDS, ignition_windows=IGNITION_WINDOWS)
```

## Module Reference

### Core Infrastructure

#### utilities.py
- `load_eeg_csv()` - Load and preprocess EEG data
- `butter_bandpass()`, `butter_highpass()` - Signal filtering
- `compute_psd_multitaper()` - Power spectral density
- `compute_iaf()` - Individual Alpha Frequency
- `plot_stacked_relpower_timeseries()` - Band power visualization

#### harmonics.py (Schumann Detection)
- `detect_schumann_spikes_wavelet()` - Morlet wavelet spike detection
- `detect_and_plot_schumann_microgrid_with_heatmaps()` - Ridge tracking
- `detect_and_plot_schumann_microgrid_with_global_tf()` - Global TF panel
- `estimate_sr_harmonics()` - Estimate harmonics from PSD peaks
- `schumann_activity_index()` - Sum of z-scored harmonic activity

#### detect_ignition.py
- `detect_ignitions_session()` - Full session ignition detection
- Returns `(output_dict, List[Tuple[int, int]])` — integer seconds (rounded)

### Spectral Analysis

#### fooof_harmonics.py (FOOOF/SpecParam Integration)
Separates periodic (oscillatory) peaks from aperiodic (1/f) background.
- `detect_harmonics_fooof()` - Single/combined PSD fit
- `detect_harmonics_fooof_multichannel()` - Per-channel or combined fits
- `extract_aperiodic_params()` - Get 1/f exponent & offset
- `match_peaks_to_canonical()` - Peak-to-harmonic matching
- `plot_fooof_fit_with_harmonics()` - Visualization
- `compare_psd_fooof()` - Before/after comparison plot

```python
# Example: FOOOF harmonic detection
from fooof_harmonics import detect_harmonics_fooof

harmonics, result = detect_harmonics_fooof(
    records=RECORDS,
    channels=['EEG.F4', 'EEG.O1', 'EEG.O2'],
    fs=128,
    f_can=(7.83, 14.3, 20.8, 27.3, 33.8),
    combine='median'
)
print("Detected harmonics:", harmonics)
print("Aperiodic exponent:", result.aperiodic_exponent)
```

#### psd_waterfall.py (Cross-Session Aggregation)
Collects ignition-window PSDs across sessions for grand visualization.
- `IgnitionPsdCollector` class with:
  - `add_session()` - Add session data (computes PSD internally)
  - `add_precomputed()` - Add pre-computed PSD data
  - `plot_grand_waterfall()` - Generate waterfall visualization
  - `plot_heatmap()` - 2D heatmap only
  - `to_dataframe()` - Export to DataFrame

```python
from psd_waterfall import IgnitionPsdCollector

collector = IgnitionPsdCollector(
    freq_range=(1, 30),
    sort_by=('sr', 7.83),  # Sort by SR power at 7.83 Hz
    normalize=True
)

for sess in SESSIONS:
    collector.add_session(
        RECORDS=sess.df, windows=sess.windows, fs=128,
        session_id=sess.id, band=(1, 45), notch=60.0,
        nperseg_sec=2.0, overlap=0.5, average='gfp'  # 'gfp', 'mean', 'median'
    )

fig, freqs, Z_all, info = collector.plot_grand_waterfall(
    title="All Sessions — Ignition PSD",
    view_preset='sr_alignment',
    heatmap_panel=True,
    sr_curtains=True
)
```

### Cross-Frequency Coupling

#### cross_frequency.py
- `pac_comodulogram_array()` - Phase-amplitude coupling
- `bicoherence_array()` - Bicoherence calculation
- `run_crossfreq_suite_records()` - Full PAC/bicoherence pipeline

#### pac_multiplexing.py
- `run_pac_vs_schumann()` - PAC dynamics vs Schumann SAI/overlap coupling

#### cross_frequency_harmonics.py
- `cf_plv()`, `mvl()`, `tort_mi()` - Cross-frequency metrics at Schumann harmonics

### Connectivity & Information Flow

#### information_flow.py
- `run_freq_granger_pdc_dtf()` - VAR/PDC/DTF analysis
- `run_transfer_entropy()` - Transfer entropy
- `run_tvar_dtf()` - Time-varying DTF

#### directed_connectivity.py, directional_coupling.py
- PDC, DTF, GC and phase/amplitude directional coupling

#### causal_routing.py
- `fit_var()` - Advanced VAR model fitting & causal inference

### Network & Graph Analysis

#### network_geometry.py
- `run_full_session_with_bands_and_exports()` - UMAP embeddings
- `run_multi_band_geometry_records()` - Multi-band graph analysis

#### network_graph_hubs.py
- Hub detection & graph community structure

#### connectome.py
- `compute_connectome_harmonics()` - Graph Laplacian harmonics
- `project_time_series_onto_harmonics()` - Spectral graph projection

### State Analysis

#### criticality.py
- `run_criticality_analysis()` - 1/f slope, DFA, avalanche statistics
- `welch_beta()` - 1/f exponent estimation
- `dfa_alpha()` - Detrended fluctuation analysis

#### microstate_segmentation.py
- `run_microstate_segmentation()` - EEG microstate analysis

#### hidden_markov.py
- `detect_schumann_bursts()` - Schumann burst detection
- `run_erp_ersp_itc_cluster_perm()` - ERP/ERSP/ITC analysis
- `run_hmm_spectral_states()` - HMM state analysis

### Advanced Signal Processing

#### temporal_dynamics.py
- `band_envelope_and_slow_phase()` - Envelope extraction
- `xcorr_lag()` - Cross-correlation lag analysis
- `lag_ci_bootstrap()` - Bootstrap confidence intervals
- `phase_lead_probability()` - Phase lead analysis

#### harmonic_locking.py
- Phase locking at Schumann harmonics with surrogate statistics

#### synchrosqueeze.py
- `ridge_in_band()`, `validate_ridge()` - Synchrosqueezed wavelet transforms
- `validate_eeg_sr_coupling()` - EEG-SR coupling validation

#### wavelet_coherence.py
- Wavelet-based time-frequency coherence

### Visualization & Utilities

#### test.py (Visualization Module)
Configuration and six-panel visualization for ignition analysis.

```python
from test import FeaturePackCfg, six_panel

# Configuration dataclass
CFG = FeaturePackCfg(
    channels=ELECTRODES,
    time_col='Timestamp',
    fs=128,
    win_sec=6.0,
    step_sec=0.01,
    spec_win=1.5,
    spec_ovl=0.95,
    sr_centers=(7.83, 14.3, 20.8),
    bw_hz=0.5,  # Can be scalar or array (one per ladder frequency)
    ladder=(7.83, 14.3, 20.8, 27.3, 33.8, 40.3, 46.8, 53.3),
    ladder_bw=0.6
)

# Generate six-panel visualization
six_panel(records, electrodes, ign_win, ign_out, ladder, CFG, session_name)
```

Color palettes: 'aquatic', 'matrix', 'spirited', 'cyberpunk', 'grand_budapest', 'blade_runner', 'mad_max', 'sunset'

#### session_metadata.py (Batch Processing)
Parses metadata from EEG filenames for multi-dataset workflows.

```python
from session_metadata import parse_session_metadata, add_metadata_to_results

# Parse metadata from filename
metadata = parse_session_metadata('data/PhySF/s10_flow.csv', 'physf')
# Returns: {'subject': 'physf_s10', 'device': 'epoc', 'context': 'flow', 'dataset': 'physf'}

# Supported datasets: 'files', 'physf', 'vep', 'mpeng1', 'mpeng2', 'insight', 'muse'
```

#### ignition_rebound.py
- `plot_bandpower()`, `plot_topomap_grid()` - Ignition/rebound visualization

## Batch Processing Workflow

Template for processing multiple datasets:

```python
from session_metadata import parse_session_metadata

RESULTS = pd.DataFrame()

for _files, dataset_name in [(FILES,'files'), (INSIGHT,'insight'), (MUSE,'muse'),
                              (PHYSF,'physf'), (VEP,'vep'), (MPENG1,'mpeng'), (MPENG2,'mpeng')]:
    for _file in _files:
        # 1. Parse metadata
        metadata = parse_session_metadata(_file, dataset_name)

        # 2. Select electrodes by device
        if metadata['device'] == 'insight':
            _electrodes = INSIGHT_ELECTRODES
        elif metadata['device'] == 'muse':
            _electrodes = MUSE_ELECTRODES
        else:
            _electrodes = ELECTRODES

        # 3. Load with device-specific parameters
        _records = utilities.load_eeg_csv(_file, electrodes=_electrodes, device=metadata['device'])

        # 4. Detect ignitions
        _ign_out, _ign_windows = detect_ignition.detect_ignitions_session(
            _records, eeg_channels=_electrodes,
            harmonic_method='fooof_hybrid',
            harmonics_hz=CANON, labels=LABELS, half_bw_hz=HALF_BW
        )

        # 5. Extract and accumulate events with metadata
        events = _ign_out['events'].copy()
        events.update(metadata)
        RESULTS = pd.concat([RESULTS, pd.DataFrame([events])], ignore_index=True)

# 6. Save batch results
RESULTS.to_csv("batch_results.csv", index=False)
```

## Output Structures

### detect_ignitions_session() Output

The `out` dictionary contains:

```python
out['events']           # DataFrame with per-event data
out['summary']          # Summary statistics
out['ignition_windows'] # List of (start, end) tuples (float seconds)
out['ignition_windows_rounded']  # List of (start, end) tuples (int seconds)
```

### Events DataFrame Columns

```python
out['events'].columns:
    # Timing
    'start_sec', 'end_sec', 'duration'

    # Per-harmonic frequencies (when labels provided)
    'sr1', 'sr1.5', 'sr2', 'sr2o', 'sr2.5', 'sr3', 'sr4', 'sr5', 'sr6'

    # Detected frequencies list
    'ignition_freqs'

    # SR envelope metrics
    'sr_z_max', 'sr_z_mean_pm5', 'sr_z_mean_post5'

    # Metadata (when added via session_metadata)
    'subject', 'device', 'context', 'dataset'
```

### Computing Ratios

```python
# Golden ratio validation
if 'sr1' in events.columns and 'sr3' in events.columns:
    events['sr3/sr1'] = events['sr3'] / events['sr1']
if 'sr1' in events.columns and 'sr5' in events.columns:
    events['sr5/sr1'] = events['sr5'] / events['sr1']
```

## FOOOF Integration

### When to Use FOOOF

Use FOOOF-based harmonic detection when:
- **Standard PSD has prominent 1/f background** obscuring harmonic peaks
- **Individual alpha frequency (IAF)** overlaps with Schumann harmonics
- **Aperiodic exponent** is of interest for state analysis

### harmonic_method Options

| Method | Description | Use Case |
|--------|-------------|----------|
| `'psd'` | Standard Welch PSD peak detection | Default, fast |
| `'fooof_session'` | FOOOF fit on full session | Stable harmonic estimates |
| `'fooof_event'` | FOOOF fit per ignition window | Event-specific harmonics |
| `'fooof_hybrid'` | Session-wide aperiodic + event peaks | Best of both |

### FOOOF Parameter Tuning

```python
detect_ignition.detect_ignitions_session(
    RECORDS,
    harmonic_method='fooof_hybrid',

    # Frequency range for FOOOF fitting
    fooof_freq_range=(5, 45),

    # Per-harmonic frequency ranges (more precise)
    fooof_freq_ranges=[[5,15], [5,15], [7,17], [12,22], [15,25], [20,30], [30,40], [35,45]],

    # Peak detection parameters
    fooof_max_n_peaks=10,          # Max peaks to fit
    fooof_peak_threshold=0.01,     # Lower = more sensitive
    fooof_min_peak_height=0.01,    # Minimum peak amplitude
    fooof_peak_width_limits=(0.1, 4),  # Peak width bounds (Hz)

    # Matching method
    fooof_match_method='power',    # 'distance' (nearest), 'power' (strongest), 'average'
)
```

## Common Patterns & Conventions

### Function Signatures
Most analysis functions follow this pattern:
```python
def run_<analysis>_<type>(
    RECORDS: pd.DataFrame,              # Main data
    ignition_windows: List[Tuple],      # [(start_sec, end_sec), ...]
    baseline_windows: List[Tuple]=None, # Optional baseline
    electrodes: List[str]=None,         # None = auto-detect EEG.* columns
    time_col: str='Timestamp',
    out_dir: str=None,                  # Export directory
    show: bool=True,                    # Show plots
    **kwargs
) -> Dict[str, object]:
```

### Channel Naming
- Raw: `'F4'`, `'O1'`, `'AF3'`
- With prefix: `'EEG.F4'`, `'EEG.O1'`
- Most functions accept either form

### Windows Format
```python
IGNITION_WINDOWS = [(178, 189), (280, 291), (561, 572)]  # (start_sec, end_sec)
# Note: detect_ignitions_session returns INTEGER tuples
```

### Return Values
Most functions return dicts with:
- `'summary'` or `'delta_table'` - Key metrics as DataFrame
- `'params'` - Parameters used
- State-specific results: `'ignition'`, `'baseline'`, `'rebound'`

## Important Constants

```python
# In utilities.py
FS = 128  # Sampling rate
ELECTRODES = ['AF3','AF4','F7','F8','F3','F4','FC5','FC6','P7','P8','T7','T8','O1','O2']
BRAINWAVES = ['Delta','Theta','Alpha','BetaL','BetaH','Gamma']
RANGES = {'Delta':[1,4],'Theta':[4,8],'Alpha':[8,12],'BetaL':[12,16],'BetaH':[16,25],'Gamma':[25,45]}

# Device-specific electrodes
INSIGHT_ELECTRODES = ['AF3', 'AF4', 'T7', 'T8', 'Pz']
MUSE_ELECTRODES = ['AF7', 'AF8', 'TP9', 'TP10']
```

## Development Guidelines

### Adding New Analysis Modules
1. Place in `lib/` directory
2. Include `infer_fs_from_records()` or similar for fs inference
3. Support both bare channel names and `EEG.` prefixed
4. Return structured dict with `summary` DataFrame
5. Include `show=True` parameter for optional plotting

### Common Helper Patterns
```python
# Infer sampling rate
def _get_fs(RECORDS, time_col='Timestamp'):
    t = RECORDS[time_col].values
    dt = np.diff(t); dt = dt[np.isfinite(dt) & (dt > 0)]
    return 1.0 / np.median(dt)

# Find channel flexibly
def find_channel_series(records, ch_name):
    patterns = ("EEG.{ch}", "eeg.{ch}", "{ch}")
    for pat in patterns:
        col = pat.format(ch=ch_name)
        if col in records.columns:
            return records[col]
    return None
```

### Band Filtering Safety
Always clamp frequencies to Nyquist:
```python
ny = 0.5 * fs
f1 = max(1e-6, min(f1, ny * 0.99))
f2 = max(f1 + 1e-6, min(f2, ny * 0.999))
```

## Testing & Validation

- Run analyses on known meditation EEG sessions
- Compare ignition windows across different detection parameters
- Validate Schumann harmonics against known values (7.83, 14.3, 20.8...)
- Use surrogate controls (phase shuffling) for statistical validation

## Dependencies

### Required
```
numpy
pandas
scipy
matplotlib
seaborn
mne (for time-frequency analysis)
networkx (for graph metrics)
scikit-learn (for UMAP, clustering)
specparam (for FOOOF harmonic detection — replaces deprecated 'fooof')
```

### Optional
- `fooof` (legacy, deprecated — use `specparam` instead)
- `entropy` (sample entropy, permutation entropy)
- `lempel_ziv_complexity`
- `statsmodels` (for Granger causality)
- `ripser` (for topological data analysis)
- `umap-learn` (for UMAP embeddings)

### Installation
```bash
pip install specparam  # Recommended for FOOOF features
# OR legacy: pip install fooof
```

## File Ignores

Per `.gitignore`:
- `data/*` - Raw EEG files
- `exports*`, `session_exports/*` - Analysis outputs
- `*.png`, `*.csv`, `*.npy` - Generated files
- `embedding_frames*` - Animation frames

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/neurokinetikz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/neurokinetikz)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
