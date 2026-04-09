## ethograph

> Never hardcode frame rates (e.g., 30 fps) or sample rates (e.g., 44100 Hz) anywhere in the codebase — not even as fallbacks. Always use actual source metadata (e.g., video.fps, audio sample rate, ImageSeries.rate) or user-specified settings. If a rate is unknown, raise an error or return None — never silently default to a hardcoded value.

## Frame Rate Guidelines

Never hardcode frame rates (e.g., 30 fps) or sample rates (e.g., 44100 Hz) anywhere in the codebase — not even as fallbacks. Always use actual source metadata (e.g., video.fps, audio sample rate, ImageSeries.rate) or user-specified settings. If a rate is unknown, raise an error or return None — never silently default to a hardcoded value.

Never hardcode a 1-second fallback for time windows, trial durations, or trial start/stop timing. If timing metadata is missing, propagate unknown timing (or raise) instead of using 1.0s placeholders.

# CLAUDE.md

## Continue

You always keep proposing things, and not implementing. Stop waiting for me to say 'yes go'.

## System prompt

---
name: python-pro
description: Write idiomatic Python code with advanced features like decorators, generators, and async/await. Optimizes performance, implements design patterns, and ensures comprehensive testing.
---

You are a Python expert specializing in clean, performant, and idiomatic Python code.

## Focus Areas
- Advanced Python features (decorators, metaclasses, descriptors)
- Performance optimization and profiling
- Design patterns and SOLID principles
- Type hints and static analysis (mypy, ruff)

## Import statements

- Remove unused imports, add missing imports, sort: stdlib → third-party → local
- Use explicit imports, never wildcard (`from x import *`)
- Never place imports inside functions/methods (only exception: avoiding circular imports)

## Philosophy for adding comments
Self-documenting code. Comments only when logic is not obvious, which should be very rare.

Never remove human-authored comments (TODO, FIXME, NOTE, explanatory comments). These represent decisions or reminders from the developer. Only remove comments that you yourself added.

## Error Handling: Fail Fast

- BUG (wrong type, missing key, None where value expected) → Let it crash
- RUNTIME CONDITION (file not found, invalid user input) → Handle gracefully
- Never wrap code in try/except that silently returns None
- Catch broad exceptions ONLY at the outermost GUI boundary

## Managing Claude.md

After making major design changes, update this file to match the current state.

## Test Files

All test and debug scripts go in `tests/`. Never leave `test_*.py` or `_test_*.py` files in the project root. Prefix with `_test_` for ad-hoc debug scripts (pytest won't discover them).

## Development Notes

Claude Code has permission to make any necessary changes to files in this repository.

## Writing docs

Don't mention indivdiuals Poppy, Freddy, Ivy anywhere in docstrings, docs.

## Project Overview

ethograph-GUI is a napari plugin for labeling start/stop times of animal movements. It integrates with ethograph, a workflow using action segmentation transformers to predict movement segments. The GUI loads NetCDF datasets, displays synchronized video/audio/ephys, and allows interactive labeling.

## Import Convention

```python
import ethograph as eto

dt = eto.open("data.nc")
dt = eto.from_datasets([ds1, ds2])
time = eto.get_time_coord(da)
data, filt = eto.sel_valid(da, kwargs)
```

## File Structure

```
ethograph/
    __init__.py               # Public API

ethograph/gui/
    plot_sources               # Plot-facing data sources + buffering (PlotSource, FileSource, XarraySource, WindowedBuffer)
    app_state.py              # Central state management (AppStateSpec + ObservableAppState)
    data_sources.py           # build_audio_source() -> FileSource
    data_loader.py            # Dataset loading: .nc, .nwb, pynapple, NWB project dirs
    pose_render.py            # Pose loading (direct NWB + movement), filtering, PoseDisplayManager
    plots_container.py        # UnifiedPanelContainer — multi-panel layout
    plots_base.py             # Abstract base class for all plots (BasePlot)
    plots_audiotrace.py       # Audio waveform (WindowedBuffer + FileSource)
    plots_spectrogram.py      # Spectrogram (SpectrogramBuffer + PlotSource)
    plots_ephystrace.py       # Ephys multichannel trace (custom pyramid buffer + FileSource)
    plots_lineplot.py         # Time-series line plot (WindowedBuffer + XarraySource)
    plots_heatmap.py          # N-dim heatmap (WindowedBuffer + XarraySource for features)
    plots_raster.py           # Spike raster plot
    plots_space.py            # 2D/3D position visualization
    plots_timeseriessource.py # Re-exports from io/time_model.py (backwards compat)
    label_drawing_mixin.py    # Shared label/changepoint drawing
    video_sync.py             # Napari video/audio synchronization (NapariVideoSync)
    video_manager.py          # Multi-camera video loading
    widget_intervals.py       # Interval navigation: browse by trial/label/sequence (IntervalNavigationWidget)
    widgets_meta.py           # Main orchestrator (MetaWidget)
    widgets_data.py           # Dataset controls (DataWidget — central orchestrator)
    widgets_io.py             # File loading, I/O controls (.nc, .nwb, .npz, pynapple folders)
    widgets_labels.py         # Label labeling interface
    widgets_navigation.py     # Navigation: Session/Trial/Label/Sequence mode ("Time slider:" combo)
    widgets_changepoints.py   # Changepoint detection + correction
    widgets_ephys.py          # Ephys controls, neurons (Kilosort/Pynapple), firing rates
    widgets_plot_settings.py  # Plot settings controls
    widgets_transform.py      # Energy envelope + noise reduction

ethograph/labels/
    intervals.py              # Interval operations, mapping loaders, find_blocks (merged from core.py)
    ml.py                     # Dense↔interval conversion, ML post-processing (stitch_gaps, purge_small_blocks, fix_endings)
    tsv_store.py              # TSV file I/O, per-trial access, validation (n_samples per-trial metadata)
    predictions.py            # Load model predictions (.npy/.pickle), confidence via 1-entropy
    crowsetta_format.py       # EthographSeq Crowsetta format (export adapter, int→string labels)
    converters.py             # Crowsetta/NWB import converters
    export.py                 # enrich_labels_df(), correct_offsets_trial()

ethograph/io/
    catalog.py                # Unified DataCatalog + DataLoader (XarrayLoader, PynappleLoader, NWBLoader)
    nwb_backend.py            # NWB traversal + combo detection: open_nwb, catalog_nwb, ComboCatalog, load_slice
    trialtree.py              # TrialTree (xr.DataTree subclass)
    time_model.py             # TimeRange, RestrictionWindow, TrialAlignment, TimeSource, SourceCollection, restriction builders
    time_sources.py           # Concrete adapters: XarrayTrialSource, PynappleSource, NWBTimeSource
    dataset.py                # dataset_to_basic_trialtree, downsample_trialtree
    feature_store.py          # Backwards-compat re-exports from catalog.py
    validation.py             # validate_datatree, extract_type_vars (delegates to catalog)
    pynapple.py               # Pynapple/NWB loading: load_nap_data, detect_trials, changepoint helpers
    restrict.py               # Re-exports from time_model.py (backwards compat, can be deleted)

ethograph/utils/
    io.py                     # Standalone I/O functions
    xr_utils.py               # sel_valid(), get_time_coord()
    sequences.py              # match_sequences, get_label_instances, get_unique_sequences
```

## Architecture

### Two Data-Source Layers

The codebase has two distinct source protocols:

**Rendering layer** (`io/plot_sources`) — what plots use to load and cache viewport data:
- **`PlotSource`** (Protocol) — `name`, `time_range`, `sampling_rate`, `identity`, `get_data(t0, t1)`
- `FileSource` — wraps any loader with `rate`/`__len__`/`__getitem__` (audioio, ephys, memmap)
- `XarraySource` — wraps `xr.Dataset`, returns time-sliced datasets from `get_data()`
- `NWBSource` — lazy HDF5 streaming from NWB TimeSeries/ImageSeries (no xarray conversion, supports remote via remfile)
- `PynappleSource` — lazy access to pynapple Tsd/TsdFrame objects via `restrict()`
- `WindowedBuffer` — viewport-aware cache. Loads wider than viewport, reloads on pan past buffer. Works with all PlotSource implementations.

**Navigation layer** (`io/time_model.py` + `io/time_sources.py`) — session-level time metadata:
- **`TimeSource`** (Protocol) — `name`, `time_range`, `sampling_rate`, `get_data(t0, t1)`
- `SourceCollection` — registry that computes `union_range`, `intersection_range`, `find_trial(t)`
- Concrete adapters: `XarrayTrialSource`, `PynappleSource`, `NWBTimeSource`, `MediaTimeSource`

`SourceCollection` only uses `time_range` metadata — it never calls `get_data()`.

**Which plots use what:**
| Plot | Source | Buffer |
|------|--------|--------|
| AudioTracePlot | `FileSource` | `WindowedBuffer` |
| SpectrogramPlot | `PlotSource` (FileSource) | `SpectrogramBuffer` (caches FFT output) |
| EphysTracePlot | `FileSource` (via buffer) | `EphysTraceBuffer` (custom: multi-resolution pyramid) |
| LinePlot | `XarraySource` | `WindowedBuffer` |
| HeatmapPlot (features) | `XarraySource` | `WindowedBuffer` |
| HeatmapPlot (envelope) | Direct loader access | Inline cache |

### TrialTree: `trialtree.py`

`TrialTree` inherits from `xr.DataTree`. Each trial is a child node with `attrs["trial"]`.

Key: `dt.trial(id)`, `dt.itrial(idx)`, `dt.trials`, `dt.trial_items()`, `dt.map_trials(fn)`, `dt.update_trial(id, fn)`, `dt.get_label_dt()`

Media & Session: Session metadata (trial timing, media file paths, FPS, stream offsets) accessed via `app_state.nwb_alignment` (NWBAlignment). For NWB sources, the source NWB file is used directly — no separate alignment.nwb is needed. For non-NWB sources (.nc, .npz), a sidecar `.ethograph/alignment.nwb` provides this metadata.

### State Management: `app_state.py`

**AppStateSpec** — type-checked spec with ~40 variables.
**ObservableAppState** — Qt signals auto-generated per variable (e.g., `current_frame_changed`). Dynamic `*_sel` attributes for xarray selections. Auto-saves to YAML.

Key signals: `trial_changed`, `restrict_window_changed`, `labels_modified`, `verification_changed`

### Time Model + Navigation: `time_model.py`

Core types in `ethograph/io/time_model.py` (canonical home, re-exported from `gui/plots_timeseriessource.py` for backwards compat):

**`TimeRange`** — immutable time interval with `union()`, `intersect()`, `contains()`, `overlaps()`.

**`TimeSource`** (Protocol) — one time-aligned data source: `name`, `time_range`, `sampling_rate`, `get_data(t0, t1) → (timestamps, values)`. Concrete adapters in `time_sources.py`: `XarrayTrialSource`, `PynappleSource`, `NWBTimeSource`.

**`SourceCollection`** — Neurosift-inspired registry of `TimeSource` objects. Provides `union_range` (full navigable extent), `intersection_range` (overlap of all sources), `session_range` (min trial start to max trial end), `sources_at(t)`, trial bookmarks (`trial_range`, `find_trial`, `trial_offset`). Built during dataset loading in `data_loader.py`, stored as `app_state.source_collection`.

**`RestrictionWindow`** — display window with mode: `"session"`, `"trial"`, `"label"`, or `"sequence"`.

`app_state.window_bounds` returns the current `RestrictionWindow.time_range` (or falls back to `trial_bounds`). `app_state.session_time_range` returns the full session extent from `SourceCollection`. All plots use `window_bounds` for x-axis limits.

**Navigation modes** (UI label: "Time slider:"):
- Session mode: slider covers entire session (inter-trial gaps navigable)
- Trial mode: standard trial navigation
- Label mode: navigate between instances of a specific label class across trials
- Sequence mode: navigate between trials matching a label sequence pattern (e.g. "1-2-3-5")

**Restriction builders** (in `time_model.py`): `build_trial_window()`, `build_label_window()`, `build_sequence_window()`, `find_closest_trial()`.

**Sequence matching** (`ethograph/utils/sequences.py`): `match_sequences()`, `get_label_instances()`, `get_unique_sequences()`.

### Unified Data Catalog + Loader: `catalog.py`

Replaces the old `type_vars_dict` pattern with two explicit abstractions:

**`DataCatalog`** — declares what's available: features, dimensions (combos), streams. Built by `catalog_from_xarray()`, `catalog_from_pynapple()`, or `catalog_from_nwb()`. The GUI creates combo boxes from `catalog.combos`. `catalog.to_type_vars_dict()` provides backwards-compat dict for existing GUI code.

**`DataLoader`** (Protocol) — backend-agnostic data access. `select(feature, selections, t0, t1) → PlotData`. Follows the `sel_valid` principle: combo selections can be overspecified, loaders ignore dimensions that don't exist on the target feature.

**`PlotData`** — source-agnostic dataclass: `time`, `data` (numpy `(T,)` or `(T,D)`), `dim_labels`, `title`, `ylabel`, `color_data`, `changepoints`, `boundary_events`. Consumed by `render_plot_data()` in `plots_lineplot.py`.

**Concrete loaders:**
- `XarrayLoader` — wraps `xr.Dataset`, delegates to `sel_valid()`. Updated on trial change via `update_ds()`.
- `PynappleLoader` — wraps raw pynapple dict. Lazy: data never copied to xarray. Uses `restrict()` for trial + time window, column indexing for dim selection.
- `NWBLoader` — wraps NWB file via `nwb_catalog` + `ComboCatalog`. Supports local and remote (remfile) access. Time slicing via rate-based indexing or `np.searchsorted` on timestamps.

**Shared column dimensions:** `_compute_shared_column_dims()` (in `catalog.py`) groups TsdFrame objects by their column values. Objects with identical columns (e.g. position & velocity both with x/y/z) share one dimension name → one combo in the GUI.

**NWB Backend** (`nwb_backend.py`): NWB traversal + combo detection in one file. `NWBCatalog` with `TimeSeriesRecord`s, `ComboCatalog` with `filter()`, `load_slice()`, `load_stacked()`.

### Alignment System: `nwb_alignment.py`

**Rule: Always create `alignment.nwb` for project directories.** The NWB wizard always produces `.ethograph/alignment.nwb` — for DANDI remote projects, local NWB projects, and non-NWB data alike. This eliminates the decision logic "source NWB vs sidecar alignment.nwb". For standalone `.nwb` file loading (no project dir), the source NWB is used directly via `_resolve_alignment`.

**Provenance**: alignment.nwb stores provenance metadata in NWB `scratch` namespace (key `ethograph_provenance`). For DANDI projects this includes `nwb_dandiset_id`, `nwb_asset_id`, `nwb_pose_keys`, etc. Read via `NWBAlignment.provenance`. Written via `write_provenance()` / `read_provenance()` in `utils/nwb.py`.

**`NWBAlignment`** reads any NWB file for session metadata. Constructed from a file path (`NWBAlignment(path)`) or an already-opened pynwb object (`NWBAlignment.from_nwb_object(nwb_obj)` — used for DANDI remote streaming). Key methods:
- `get_stream_rate(stream, device)` — read `.rate` from any ImageSeries
- `resolve_media_path(trial, stream, device, fallback_folder)` — try ImageSeries path → NWB-relative → fallback folder + filename. URL-aware: returns URLs directly, prefers local fallback_folder copy if available.
- `stream_offset_for_trial(trial, stream, device)` — trial-relative offset derived from ImageSeries timing

**Priority order** (`_resolve_alignment` in `data_loader.py`, used for standalone file loading only): For `.nwb` sources: source NWB → sidecar alignment.nwb → sidecar metadata TSV. For other sources: sidecar alignment.nwb → sidecar metadata TSV. Project directories bypass `_resolve_alignment` — `_load_nwb_project` reads alignment.nwb directly.

**Path fallback**: ImageSeries stores original paths (or URLs for DANDI). If files move, `resolve_media_path` extracts the filename and joins with a user-specified fallback folder (`video_folder`, `audio_folder`, etc.).

**`build_nwb_from_trial_table`** (`utils/nwb.py`) creates ImageSeries for ALL streams via `sync_acquisition_for_streams(nwbfile, stream_rates)`. Takes `stream_rates: dict[str, float]` — no hardcoded FPS values.

**`create_alignment_from_streams`** (`utils/nwb.py`) — flexible alignment creation accepting per-trial or session-wide files, optional `provenance` dict. Used by the NWB wizard to create alignment.nwb.

### Data Loading: `data_loader.py`

The GUI supports loading `.nc` (NetCDF), `.nwb`, `.npz`, pynapple folders, and NWB project directories. Dispatch in `data_loader.py`:
- `.nc` → `eto.open()` + `catalog_from_xarray()`. If `.nc` has `nwb_source` attr, also attaches a `PynappleLoader` for lazy NWB features.
- `.nwb` → `catalog_from_nwb()` + `NWBLoader` (direct HDF5 slicing via `ComboCatalog`).
- `.npz`/folder → `load_nap_data()` + `catalog_from_pynapple()` + `PynappleLoader`.
- NWB project dir (has `.ethograph/alignment.nwb`) → reads provenance from alignment.nwb to determine source. Local NWB: loads via pynapple. DANDI remote: opens via `open_nwb_dandi()` + `NWBLoader` (direct HDF5 slicing). Alignment always from alignment.nwb.

`load_dataset()` returns `(dt, all_labels_df, catalog)` where `catalog` is a `DataCatalog`.

### Pose Rendering: `pose_render.py`

Two loading paths for pose data:
- **File-based** (DLC, SLEAP, etc.): `load_pose_from_file()` via `movement.io.load_dataset` + `ds_to_napari_layers`
- **NWB-based**: `load_pose_from_nwb_direct()` reads `PoseEstimationSeries.data` and `.confidence` directly via lazy HDF5 slicing — no xarray/movement intermediate

`PoseRenderData` is the unified result type. `apply_confidence_filter()` and `apply_keypoint_filter()` work on the `data_not_nan` mask. `PoseDisplayManager` orchestrates display via `shown` mask — filtering never recreates layers.

The NWB wizard stores `nwb_pose_keys` (e.g. `["LeftCamera", "RightCamera"]`) in the project config. At render time, `PoseDisplayManager._load_pose_for_camera()` maps camera index → pose key → direct NWB loading.

### Plot System

All plots inherit `BasePlot` (pyqtgraph `PlotWidget`): time marker, x-axis range management, click handling, axes locking.

`UnifiedPanelContainer` holds all panels in a `QSplitter`, links x-axes, manages panel visibility.

### Rendering Sources & Streaming (`io/plot_sources`)

The rendering layer provides **lazy, viewport-aware data streaming** without converting to xarray. All sources implement the `PlotSource` protocol and work with `WindowedBuffer` for efficient caching.

**Core Pattern:**
```python
# Create a source (lazy, no data loaded yet)
source = NWBSource("data.nwb", "acquisition/video_cam-1")

# Wrap in buffer for viewport caching
buffer = WindowedBuffer(buffer_multiplier=5.0)
buffer.set_source(source)

# Viewport pan/zoom triggers lazy load
data = buffer.get(t0=10.0, t1=20.0)  # SampleSlice(timestamps, values)
```

**Source Types:**
- `FileSource` — audioio, ephys, memmap loaders (existing)
- `XarraySource` — xr.Dataset slicing (existing)
- `NWBSource` — **NEW** — Direct HDF5 streaming from NWB TimeSeries/ImageSeries
  - Lazy metadata read (no file open until needed)
  - Supports explicit timestamps or rate-based timing
  - Enabling point for remfile/remote streaming
- `PynappleSource` — **NEW** — Direct pynapple Tsd/TsdFrame access
  - Wraps `restrict()` for time slicing
  - No xarray intermediate

**Why no xarray conversion?**
- `nwb_import.py` currently converts NWB → xarray → DataLoader. This is memory-heavy and slow.
- New sources are file-aware: HDF5 for NWB, in-memory for pynapple.
- `WindowedBuffer` caches only the viewport window, not the full dataset.
- Enables future remote streaming (remfile) without refactoring.

**Usage in DataLoader:**
Future refactor to pass `NWBSource` / `PynappleSource` directly to plots instead of converting to xarray. Catalog/metadata layer (`DataCatalog`) remains unchanged.

### Video Sync: `video_sync.py`

`NapariVideoSync`: `frame_to_time(frame)` / `time_to_frame(t)` with `time_offset`. All widgets use these — never raw `frame / fps`.

### Labels

**Storage:** TSV file (`{name}_labels.tsv`) alongside the `.nc`. Columns: `onset_s, offset_s, labels (int), individual, trial, human_verified, changepoint_corrected, prediction_source, n_samples`. The `n_samples` column stores per-trial sample count for dense conversion. Label names managed centrally in `mapping.txt`.

**Module structure:** `intervals.py` has interval operations + mapping loaders + `find_blocks`. `ml.py` has dense↔interval conversion + ML post-processing (`stitch_gaps`, `purge_small_blocks`, `fix_endings`). Old `core.py` merged into `intervals.py`; old `dense.py` renamed to `ml.py`.

**In-memory:** `app_state._all_labels_df` (all trials), `app_state.label_intervals` (current trial view). Per-trial metadata stored as columns, not a separate dict.

**Predictions:** Per-trial `.npy`/`.pickle` files in prediction folders. Shape `(T, n_classes)` → confidence via `1 - normalized_entropy`, labels via `argmax`. Confidence stays in memory for GUI overlay, never stored in our format.

**Crowsetta interop:** `EthographSeq` format registered at import time. Export adapter converts int→string labels via mapping for sharing. Internal storage stays integer-based.

**Migration:** Old `.nc` files with embedded labels auto-migrate to TSV on first load.

### Widget Orchestration

`MetaWidget` creates all widgets and wires signals. `DataWidget` is the central orchestrator — handles trial changes, plot updates, video/audio loading.

**Signal flow:** `NavigationWidget` → `trial_changed` → `DataWidget.on_trial_changed()` → updates everything.

### Neuron Loading (Kilosort / Pynapple)

Two loading paths, both producing a `nap.TsGroup` + cluster table:
- **Kilosort folder**: loads `.npy` files, `cluster_info.tsv`, raw `.dat` trace. Full features (probe map, spike waveforms, raster).
- **Pynapple file** (`.npz`/`.nwb`): loads via `nap.load_file()`, extracts `data["units"]` TsGroup. Raster-only (no raw traces, no probe map). TsGroup metadata columns populate the cluster table.

State: `app_state.neurons_path` (was `kilosort_folder`), `app_state.has_neurons` (was `has_kilosort`). `EphysWidget._neurons_source` is `"kilosort"` or `"pynapple"`.

### Kilosort Channel Mapping

Two index spaces: **site index** (0..n_sites-1, indexes `channel_positions.npy`) vs **hardware channel** (from `channel_map.npy`, can exceed n_sites). `cluster_info.tsv` `ch` column = hardware channel. Always index `channel_positions` by site index.

### Changepoint Correction

Bridge pattern: intervals→dense→correct→intervals. Kinematic CPs stored as dense `int8` arrays. Audio CPs stored as onset/offset float pairs (compact at 44kHz).

---

## Dataset Structure

- NetCDF with trials. Time coords: `time`, `time_aux`, etc. (any containing 'time')
- Variables with `type='features'` for feature selection
- Media/session metadata: for NWB sources, read from the source NWB directly; for non-NWB sources, from `.ethograph/alignment.nwb`
- Labels: stored in `_labels.tsv` (not inside `.nc`). Legacy `.nc` labels auto-migrate on first load.

## Roadmap

### Testing
Add tests for changepoints, plot content verification, model predictions loaded.

### Integration with models
Audio: vocalpy/vak. Video: DLC2Action.

### Labels I/O
TODO: Crowsetta export (CSV, Audacity, BORIS, Raven). Interval-native changepoint correction (eliminate dense bridge).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Akseli-Ilmanen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Akseli-Ilmanen)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
