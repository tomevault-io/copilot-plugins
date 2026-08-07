## nbdev-upc-aidl-iemocap-datasets

> > **ATTENTION AGENT:** Welcome to the `nbdev-upc-aidl-iemocap-datasets` workspace. Please read this file carefully before performing any modifications, creating new files, or writing implementation plans.

# 🤖 Agent Guidelines: Readme & Development Instructions

> **ATTENTION AGENT:** Welcome to the `nbdev-upc-aidl-iemocap-datasets` workspace. Please read this file carefully before performing any modifications, creating new files, or writing implementation plans.

---

## 🧭 Crucial Context & Navigation

All architectural details, data flow diagrams, schemas, and requirements are fully defined in the primary documentation of this repository.

1. **Read the README first**: 
   Before writing any code or proposing implementation plans, read the full [README.md](file:///Users/diego/development/gofordiego/nbdev-upc-aidl-iemocap-datasets/README.md). It outlines:
   - The remote Cloudflare R2 bucket URLs and caching rules.
   - The SQLite database schema (`iemocap_dataset`) for raw audio retrieval.
   - The Parquet audio chunk schemas.
   - Detailed specifications for the PyTorch `Dataset` (`AudioChunksDataset`), features extraction (Log-Mel Spectrogram), labels tensor formatting, and notebook helper methods.

2. **Source Code Location & `nbdev` Rules**:
   - This project uses **`nbdev`** (Notebook-Driven Development). Note of nbdev3 latest breaking changes for instance, no more settings.ini and pyproject.toml is used instead.
   - DO NOT ATTEMPT to produce nbdev related files without running the proper nbdev commands.
   - ⚠️ **DO NOT write or modify Python files in `src/` directly.** 
   - All source code, module exports, and unit tests must be written in Jupyter Notebooks (`.ipynb`) located in the root or designated notebooks folder.
   - Changes are exported to `src/` automatically by executing:
     ```bash
     uv run nbdev-export
     ```
   - Don't chage nbdev-export to the old nbdev_export underscore command. Remember we're running in `uv`.

3. **Dependency and Environment Management**:
   - The package manager is **`uv`**.
   - Do not use `pip` directly unless inside a `uv` wrapper or specified otherwise.
   - Use `uv sync` to keep the virtual environment up to date and `uv run` to execute commands.

4. **Git Staging Rules**:
   - Never add files to git staging (`git add`). Only the user reviewer should perform staging and commit actions.

---

## 🛠️ Completed Tasks & Implementations

The dataset factory and PyTorch dataset modules have been successfully implemented:

- [x] **Factory Initialization**: Initialize cache at `~/.cache/upc-aidl-iemocap-datasets/`, download the JSON chunks manifest, download the SQLite DB with progress bar & ETA estimation (skipping download if already present), and cache the Parquet audio chunk files.
- [x] **Listing & Refreshing**: Implement `get_dataset_audio_chunk_groups(refresh=True/False)` returning a pandas DataFrame of the manifest.
- [x] **Dataset building**: Build `AudioChunksDataset` with custom spectrogram parameters (`n_fft`, `win_length`, `hop_length`, `n_mels`).
- [x] **Feature Extraction Pipeline**: In `__getitem__`, perform SQLite lookups, decode `audio_bytes` via `soundfile`, slice, conditionally pad up to chunk_threshold_seconds * sample_rate` samples, and calculate Log-Mel Spectrogram tensors using `librosa`.
- [x] **IPython Playback**: Implement the `play_item_audio` and `get_item_audio_bytes` methods for interactive audio review in notebooks.

Always adhere to these guidelines to ensure consistency, clean exports, and robust test coverage.
erage.

---
> Source: [gofordiego/nbdev-upc-aidl-iemocap-datasets](https://github.com/gofordiego/nbdev-upc-aidl-iemocap-datasets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
