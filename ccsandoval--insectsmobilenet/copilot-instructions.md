## insectsmobilenet

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

InsectsMobileNet is a butterfly species classification project using MobileNetV3Small (transfer learning) with a Streamlit web interface. The current focus is on 5 Colombian butterfly species, with a species registry of 90+ targets. Training data is scraped from the iNaturalist API.

## Environment Setup

```bash
conda env create -f environment.yml
conda activate semillero
```

Key dependencies: Python 3.12.12, TensorFlow 2.19.1, Streamlit ≥1.35.

## Common Commands

**Train the model:** Run `main.ipynb` in Jupyter. It auto-generates the dataset split from `./Insectos/` then trains for up to 40 epochs with early stopping.

**Run the web app:**
```bash
streamlit run streamlit_app.py
```

**Scrape images for new species:**
```bash
python scripts/scrapeSpeciesImages.py
```
This reads `species_list.json` and downloads up to 90 images per species from iNaturalist, skipping species already marked `scraped: true`.

**Regenerate dataset splits:**
```bash
python scripts/generateDataset.py
```
Scans `./Insectos/` and rebuilds `./dataset/{train,test,validate}/` with a 70/20/10 split.

## Architecture

### Data Flow
```
iNaturalist API → scrapeSpeciesImages.py → Insectos/{species}/
                                        → generateDataset.py → dataset/{train,test,validate}/
                                                             → main.ipynb (training)
                                                                       → outputs/model.keras
                                                                       → streamlit_app.py
```

### Model (defined in `main.ipynb`)
- Base: MobileNetV3Small pretrained on ImageNet (frozen)
- Custom head: GlobalAveragePooling2D → Dense(512) → Dense(256) → Dense(128) → Softmax
- Each Dense layer has BatchNormalization and increasing Dropout (0.30 → 0.40 → 0.50)
- Input: 320×320 RGB; trained with Adam (lr=1e-3), categorical crossentropy, class weighting

### Key Files
- `main.ipynb` — full ML pipeline: dataset generation, training, evaluation, Grad-CAM
- `streamlit_app.py` — 3-tab UI (detection, dataset explorer, model metrics); UI is in Spanish
- `scripts/modelGradCam.py` — `computeGradcam()` and `overlayHeatmap()` using the `conv_1` layer
- `scripts/scrapeSpeciesImages.py` — iNaturalist API client with 0.25s rate limiting
- `species_list.json` — registry of all 90 target species with `scraped` status flags

### Data Directories (git-ignored)
- `Insectos/` — raw downloaded images, one folder per species (snake_case names)
- `dataset/` — processed train/test/validate splits
- `outputs/` — saved model (`model.keras`) and generated visualizations

## Adding a New Species
1. Add entry to `species_list.json` with `"scraped": false`
2. Run `scrapeSpeciesImages.py` to download images into `Insectos/{species_name}/`
3. Run `generateDataset.py` (or re-run the notebook) to rebuild splits
4. Retrain via `main.ipynb`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CCSandoval) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
