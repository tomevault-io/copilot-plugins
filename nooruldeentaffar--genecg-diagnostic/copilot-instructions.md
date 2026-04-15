## genecg-diagnostic

> * **Goal:** Multi-label classification of 12-lead ECG images into 65 SNOMED-CT diagnostic categories.

# 🩺 Project Context: ECG-ViT (Lead Detection & Multi-Label Diagnosis)

## 1. Project Overview
* **Goal:** Multi-label classification of 12-lead ECG images into 65 SNOMED-CT diagnostic categories.
* **Architecture:** Two-Stage Pipeline.
    * **Stage 1 (Pre-processing):** Mapping PTB-XL labels to SNOMED via R (Complete).
    * **Stage 2 (Object Detection):** YOLOv8 model to detect and crop 12 individual leads from a 3x4 grid.
    * **Stage 4 (Classification):** Vision Transformer (ViT) for final multi-label diagnosis.

## 2. Infrastructure & Data Layout
* **Storage:** Data is on an **External SSD**.
* **Raw Data Path:** `data/Raw/GenECG/Dataset_A_ECGs_without_imperfections/`
* **Folder Structure:** Images are stored in "buckets" (e.g., `.../00000/`, `.../01000/`).
* **Processed Data Path:** `data/Processed/YOLO_Labels/`

## 3. Rules of Engagement (Instructions for Gemini)

### 🔴 PROTOCOL: FILE_ACCESS
- **Recursive Search:** Always use `rglob('*.png')` or `**/*.png` to account for the bucket subfolders (00000, 01000, etc.).
- **LFS Verification:** Before processing an image, verify its size is > 50KB to ensure it is not a 130-byte Git LFS pointer.
- **External Drive:** Ensure all write operations target the project root on the SSD, not the internal `/Users/` directory.

### 🟠 PROTOCOL: STAGE 2 (YOLO Label Generation)
- **Heuristic:** Use a 3x4 grid assumption to generate YOLO `.txt` labels for the 12 leads.
- **Classes:** - 0-2: Leads I, II, III (Column 1)
  - 3-5: aVR, aVL, aVF (Column 2)
  - 6-8: V1, V2, V3 (Column 3)
  - 9-11: V4, V5, V6 (Column 4)
- **Target:** Produce 1,000 labels initially for testing.

### 🟡 PROTOCOL: STAGE 4 (ViT Model)
- **Inputs:** Cropped lead images from Stage 2.
- **Outputs:** 65 SNOMED classes.
- **Loss Function:** `BCEWithLogitsLoss`.

## 4. Current Progress Status
- [x] **Stage 1:** SNOMED Labeling (Parquet file generated).
- [x] **Data Acquisition:** Dataset A images confirmed on SSD (~100KB each).
- [ ] **Current Task:** Run `src/Data_pipeline/run_mass_label_generation.py` to generate the first 1,000 YOLO labels.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NooruldeenTaffar)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NooruldeenTaffar)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
