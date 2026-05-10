## arabic-itsm-classification

> This file is the living memory for Claude Code sessions on this project.

# CLAUDE.md — Project Memory & Tracking
# arabic-itsm-classification

This file is the living memory for Claude Code sessions on this project.
Update it continuously as decisions are made, experiments are run, and milestones are reached.
It will serve as a primary source for the university post-project documentation report.

---

## Assistant Operating Baseline

- Follow repository-local instructions in `AGENTS.md` as the primary operating policy.
- Keep experiment reporting reproducible by anchoring claims to notebook outputs and files under `results/`.
- Archive each full pipeline analysis using naming: `analysis_nb<NN>_<env>_<date>.md` (env: `local_cpu`, `local_gpu`, `kaggle_gpu`).

---

## Project Identity

| Field            | Value                                                                 |
|------------------|-----------------------------------------------------------------------|
| **Title**        | Cloud-Based ITSM Ticket Classification Platform Using Fine-Tuned Transformer Models |
| **Author**       | Mohamed A. Elbaz                                           |
| **Supervisor**   | Dr. Eman E. Sanad, Assistant Professor of IT, FCAI, Cairo University |
| **Degree**       | Professional Master's in Cloud Computing Networks                    |
| **Institution**  | Faculty of Computers and Artificial Intelligence, Cairo University    |
| **Date Started** | February 2026                                                        |
| **Repo**         | arabic-itsm-classification                                           |
| **Dataset Repo** | [bazokhan/arabic-itsm-dataset](https://github.com/bazokhan/arabic-itsm-dataset) |

---

## Dataset Summary

| Property          | Value                                                   |
|-------------------|---------------------------------------------------------|
| Source (GitHub)   | https://github.com/bazokhan/arabic-itsm-dataset         |
| Source (HF)       | https://huggingface.co/datasets/albaz2000/arabic-itsm-dataset |
| Size              | 10,000 tickets                                          |
| Dialect           | Egyptian Arabic (عامية مصرية) with EN code-mixing       |
| Generation        | LLM-generated (Gemini), validated programmatically      |
| L1 categories     | 6 (Access, Network, Hardware, Software, Security, Service) |
| L2 categories     | 16                                                     |
| L3 categories     | 48                                                     |
| Labels            | category_level_1/2/3, priority (1-5), sentiment        |
| Format            | CSV + JSONL                                             |

---

## Architecture Decisions

### ADR-001 — Primary Model: MarBERTv2
- **Date**: February 2026
- **Decision**: Use `UBC-NLP/MARBERTv2` as the primary encoder
- **Rationale**: Best-in-class for Egyptian colloquial Arabic; pretrained on Twitter (dialect-heavy, noisy); validated Feb 2025 benchmarks show 64–85% F1 on Egyptian classification tasks; low compute cost vs ByT5
- **Alternatives considered**: CAMeLBERT (good but MSA-biased), AraBERTv2 (MSA-only), ByT5 (excellent noise handling but high compute)
- **Reference**: `docs/model_recommendation.md`
- **Status**: Accepted

### ADR-002 — Classification Strategy: Multi-Head, Start with L1
- **Date**: February 2026
- **Decision**: Train separate classification heads for L1 (6 classes), L2 (16 classes), L3 (48 classes), and priority (5 classes). Prioritize L1 first, then extend using the same pipeline structure.
- **Rationale**: Flat multiclass gets sparse at deeper levels after deduplication. Hierarchical modeling mirrors ITSM routing logic and keeps interpretation clean.
- **Status**: Accepted

### ADR-003 — Framework: HuggingFace Transformers + PyTorch
- **Date**: February 2026
- **Decision**: Use `transformers` + `torch` with `Trainer` API for fine-tuning
- **Rationale**: Native support for MarBERTv2; standard in academic NLP work; easy to export to ONNX/TorchScript for deployment
- **Status**: Accepted

### ADR-004 — Data Split: Stratified 70/15/15
- **Date**: February 2026
- **Decision**: Stratified split on L1 label — 70% train, 15% validation, 15% test
- **Rationale**: Maintains class distribution across splits; 1,500 test samples sufficient for reliable metric estimation; validation set sized for hyperparameter search
- **Status**: Accepted

### ADR-005 — Dataset Deduplication Applied in Preprocessing, Not at Source
- **Date**: February 2026
- **Decision**: Remove 451 exact duplicate (title_ar, description_ar) pairs during preprocessing in Notebook 02, not by modifying the source dataset CSV on GitHub/HuggingFace.
- **Rationale**: Duplicates create data leakage when identical texts land in both train and test splits, artificially inflating all metrics. Fixing in preprocessing rather than at source preserves the raw dataset as a stable, citable artefact while keeping the transformation explicit and reproducible. This is standard ML practice and must be documented in the methodology chapter of the thesis.
- **What was NOT changed**: The source dataset (`bazokhan/arabic-itsm-dataset`) is unchanged. The L2 (16 classes) and L3 (48 classes) counts were found to be correct — the dataset README had the wrong numbers (14/31). The README of the dataset repo should be updated to reflect the true counts.
- **Impact**: Post-dedup dataset size is ~9,549 tickets (down from 10,000). Train/val/test splits are recomputed from the deduplicated base.
- **Status**: Accepted — applied in Notebook 02 cell `162d9cc6`

### ADR-006 — PyTorch Installation Must Specify CUDA Wheel Explicitly
- **Date**: February 2026
- **Decision**: Do not use bare `pip install torch`. Always install via the PyTorch CUDA index URL: `pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121`
- **Rationale**: Bare `pip install torch` on Windows silently installs the CPU-only wheel (`torch 2.x+cpu`). This caused a complete training failure in EXP-002 — `torch.cuda.is_available()` returned `False` despite a functional RTX 3050 GPU being present. The error is silent and the model trains for hours on CPU before the failure is detectable.
- **Consequences**: `requirements.txt` now contains a comment block instead of a bare `torch` entry. All new environment setups must follow the two-step procedure: (1) update NVIDIA driver ≥550, (2) install CUDA wheel.
- **Status**: Accepted

### ADR-007 — Storage-Aware Model Retention Policy
- **Date**: February 2026
- **Decision**: Keep only academically significant checkpoints, not every run. Always preserve at least:
  - last failed diagnostic model (if it explains a major issue),
  - best validated checkpoint per experiment stage,
  - final deployment candidate.
- **Rationale**: Balances reproducibility and thesis traceability with limited local storage on personal hardware.
- **Operational rule**: Compress/archive older intermediate checkpoints and keep metrics/config snapshots even when raw weights are removed.
- **Status**: Accepted

### ADR-008 — Statistical Significance Is Required for Final Claims
- **Date**: February 2026
- **Decision**: Add a dedicated statistical testing stage (bootstrap confidence intervals + McNemar test) before asserting superiority over best baseline.
- **Rationale**: Run-002 improvements are real but moderate; significance and uncertainty must be reported academically.
- **Status**: Accepted

### ADR-009 — Kaggle GPU for L3+ Training (GPU Constraint Rationale)
- **Date**: February 2026
- **Decision**: Use Kaggle free-tier T4 GPU for all training involving L3 heads or more. Local RTX 3050 (4 GB VRAM) handles L1-only and L1+L2 joint training but is not used for L3+ due to time and reliability constraints with the larger task graph.
- **Rationale**: Kaggle provides 2× Tesla T4 (30 GPU hrs/week), sufficient for 10-epoch L3+ runs (~13 min each). Local GPU is reserved for fast iteration (L1, L2). This split also provides a clean environment separation: local = development, Kaggle = production training.
- **Operational rule**: Always push code changes to GitHub before running on Kaggle. Kaggle notebook clones from `main` branch.
- **Status**: Accepted

---

## Experiment Log

| ID | Date | Task | Model | Epochs | LR | Batch | Val F1 (macro) | Test F1 (macro) | Notes |
|----|------|------|-------|--------|-----|-------|----------------|-----------------|-------|
| EXP-001a | 2026-02-24 | L1 baseline | TF-IDF + LR (word+char) | — | — | — | 0.8852 | 0.8748 | 3.1s train, 0.24ms infer |
| EXP-001b | 2026-02-24 | L1 baseline | TF-IDF + LinearSVC (word+char) | — | — | — | 0.8817 | **0.8840** | 6.1s train, 0.23ms infer — best baseline |
| EXP-001c | 2026-02-24 | L1 baseline | TF-IDF + Naive Bayes (word) | — | — | — | 0.8628 | 0.8526 | 0.3s train, 0.04ms infer |
| EXP-002 | 2026-02-24 | L1 | MarBERTv2 | 3 (early stop) | 2e-5 | 32 | 0.1316 | 0.1236 | **FAILED** — CPU-only training (`torch+cpu` build). Mode collapsed to "Service". Archived in `docs/analysis_run_001.md` |
| EXP-003 | 2026-02-24 | L1 | MarBERTv2 (GPU) | 4 (best at epoch 2) | 2e-5 | 32 | **0.8938** | **0.8910** | **SUCCESS** — CUDA+FP16 run. Accuracy 0.8904, infer 9.2ms/sample. Detailed analysis: `docs/analysis_nb04_local_gpu_2026-02-24.md` |
| EXP-004 | 2026-02-24 | L3 only | MarBERTv2 (Kaggle T4) | 10 (best at epoch 6) | 1e-5 | 16 | **0.7924** | TBD | **SUCCESS** — Kaggle single-task L3 run. 48 classes. Checkpoint: `marbert_l3only_kaggle/`. Analysis: `docs/analysis_kaggle_l3only_kaggle_gpu_2026-02-24.md` |
| EXP-007 | TBD | L1+L2+L3 | AraBERTv2-base (Kaggle T4) | 10 | 1e-5 | 16 | TBD | TBD | **PENDING** — Encoder ablation vs EXP-006a. Only variable: `aubmindlab/bert-base-arabertv02` instead of MARBERTv2. Notebook: `kaggle_train_arabert_l1l2l3_arabic_itsm_classification.ipynb` |

---

## Milestones

- [x] Dataset created and published (`arabic-itsm-dataset`)
- [x] Model selection documented (`docs/model_recommendation.md`)
- [x] Repo scaffolded with notebooks, src, configs
- [x] Notebook 01: Data inspection complete (with outputs)
- [x] Notebook 02: Data preparation pipeline complete
- [x] Notebook 03: Baseline models run and results recorded (LinearSVC best: 88.40% macro-F1 on dedup split)
- [x] Notebook 04: MarBERTv2 first run complete (failed — CPU env, documented in EXP-002)
- [x] Notebook 05: Evaluation run complete (on failed model — results invalid, to be rerun after GPU fix)
- [x] Infrastructure issue diagnosed and documented: CPU-only PyTorch wheel, driver update required (ADR-005)
- [x] NVIDIA driver updated to ≥550 + PyTorch reinstalled with CUDA 12.1 (prerequisite for EXP-003)
- [x] Notebook 04: MarBERTv2 GPU retraining — EXP-003 → `marbert_l1_best/`
- [x] Notebook 05: Re-evaluation after successful GPU training
- [x] Notebook 06: Statistical significance testing (McNemar + Bootstrap)
- [x] Notebook 07: L1+L2 joint training — `marbert_l2_best/` (code written + executed locally)
- [x] Notebook 08: L1+L2+L3 joint training — code written; training pending Kaggle run → `marbert_l1_l2_l3_best/`
- [x] Notebook 09: Full multi-task (5 tasks) — code written; training pending Kaggle run → `marbert_multi_task_best/`
- [x] Kaggle L3-only run — EXP-004, val F1=0.7924 → `marbert_l3only_kaggle/`
- [ ] Run `kaggle_train_l1l2l3_arabic_itsm_classification` notebook on Kaggle
- [ ] Run `kaggle_train_multitask_arabic_itsm_classification` notebook on Kaggle
- [ ] Fill analysis stubs after Kaggle runs complete
- [ ] University documentation draft started
- [ ] Web demo prototype (cloud deployment)

---

## Key File Locations

| Artifact                        | Path                                              |
|---------------------------------|---------------------------------------------------|
| Dataset CSV                     | https://raw.githubusercontent.com/bazokhan/arabic-itsm-dataset/master/dataset_clean.csv |
| Dataset JSONL                   | https://raw.githubusercontent.com/bazokhan/arabic-itsm-dataset/master/dataset_clean.jsonl |
| Taxonomy                        | https://raw.githubusercontent.com/bazokhan/arabic-itsm-dataset/master/taxonomy_itsm_v1.json |
| Model config                    | `configs/model_config.yaml`                       |
| Data config                     | `configs/data_config.yaml`                        |
| L1 checkpoint (1 head)          | `models/marbert_l1_best/`                         |
| L1+L2 checkpoint (2 heads)      | `models/marbert_l2_best/`                         |
| L3-only checkpoint (1 head)     | `models/marbert_l3only_kaggle/`                   |
| L1+L2+L3 checkpoint (3 heads)   | `models/marbert_l1_l2_l3_best/` *(pending Kaggle)* |
| Multi-task checkpoint (5 heads) | `models/marbert_multi_task_best/` *(pending Kaggle)* |
| AraBERTv2 L1+L2+L3 checkpoint (3 heads) | `models/arabert_l1_l2_l3_best/` *(pending EXP-007)* |
| Results (metrics)               | `results/metrics/`                                |
| Results (figures)               | `results/figures/`                                |

---

## Model Registry

| Checkpoint | Tasks | Heads | Environment | Status |
|---|---|---|---|---|
| `marbert_l1_best/` | L1 | 1 | Local GPU (RTX 3050) | Done — Notebook 04, EXP-003, 89.1% F1 |
| `marbert_l2_best/` | L1+L2 | 2 | Local GPU (RTX 3050) | Done — Notebook 07, EXP-003b, 89.31%/86.57% F1 |
| `marbert_l3only_kaggle/` | L3 only | 1 | Kaggle GPU (T4) | Done — Kaggle NB (EXP-004), val F1=0.7924 |
| `marbert_l1_l2_l3_best/` | L1+L2+L3 | 3 | Kaggle GPU (T4) | Pending — new Kaggle NB (Notebook 08) |
| `marbert_multi_task_best/` | All 5 | 5 | Kaggle GPU (T4) | Pending — new Kaggle NB (Notebook 09) |
| `arabert_l1_l2_l3_best/` | L1+L2+L3 | 3 | Kaggle GPU (T4) | Pending — EXP-007, `kaggle_train_arabert_l1l2l3_arabic_itsm_classification.ipynb` |

---

## Notes for University Documentation Report

*(Append notes here as the project progresses — these will feed directly into the final report)*

### Chapter 2 — Literature Review Angles
- Arabic NLP challenges: dialect diversity, orthographic variation, code-mixing
- ITSM automation: ticket routing, SLA compliance, priority prediction
- Transformer models for Arabic: AraBERT → CAMeLBERT → MarBERT → MarBERTv2 evolution
- Synthetic data for low-resource languages: LLM-generated datasets for domain adaptation

### Chapter 3 — Methodology Notes
- Dataset: synthetic LLM-generated, 10,000 tickets, Egyptian Arabic, 3-level taxonomy
- Preprocessing: arabert normalization (no diacritics, alif normalization), truncation at 128 tokens
- Model: MarBERTv2 encoder + linear head per task
- Training: AdamW optimizer, linear warmup + decay, early stopping on val macro-F1
- Evaluation: macro-F1, per-class F1, confusion matrix, latency measurement

### Chapter 4 — Results Placeholder
*(Fill in after experiments)*

### Chapter 5 — Deployment Architecture Notes
- Model served via FastAPI
- Cloud: containerized with Docker
- Input: Arabic ticket title + description → JSON response {l1, l2, priority, confidence}

---

## Known Issues & Workarounds

| Issue | Platform | Root Cause | Fix |
|-------|----------|------------|-----|
| `mlflow ui` crashes with `OSError: [WinError 10022]` | Windows | MLflow's default multi-worker uvicorn tries to share sockets across processes — not supported on Windows | Run `mlflow ui --workers 1` |
| `torch.cuda.is_available()` returns `False` despite NVIDIA GPU present | Windows | `pip install torch` installs the CPU-only wheel (`torch 2.x+cpu`) by default | (1) Update NVIDIA driver to ≥550 from nvidia.com; (2) `pip uninstall torch torchvision torchaudio -y`; (3) `pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121` |
| `FutureWarning: torch.cuda.amp.GradScaler` deprecated | All | Old API path changed in PyTorch 2.x | Use `torch.amp.GradScaler(device_type, enabled=flag)` — fixed in Notebook 04 cell `79e8d5a5` |
| `UserWarning: pin_memory set but no accelerator found` | CPU | `pin_memory=True` hardcoded in DataLoader | Make conditional: `pin_memory=(DEVICE.type == 'cuda')` — fixed in Notebook 04 cell `89917262` |
| MarBERT LOAD REPORT shows UNEXPECTED keys | All | Normal — MLM/NSP head weights from pretraining are discarded when loading bare encoder | Expected behaviour, safe to ignore. Documented in Notebook 04 cell `91ecb08a` |

---

## Session Log

| Date       | Session Summary |
|------------|-----------------|
| 2026-02-24 | Initial repo scaffold: README, CLAUDE.md, folder structure, configs, src package, 5 demo notebooks created |
| 2026-02-24 | Fixed all dataset references to use GitHub/HuggingFace URLs (no local clone required). Rewrote model_recommendation.md as full academic comparative analysis. |
| 2026-02-24 | Diagnosed MLflow WinError 10022 — fixed with `--workers 1` flag |
| 2026-02-24 | Fixed FutureWarning in Notebook 04: `torch.cuda.amp` → `torch.amp` (device-agnostic API). Documented MarBERT UNEXPECTED keys as normal behaviour. |
| 2026-02-24 | All 5 notebooks executed. Baselines complete. MarBERT failed (EXP-002) — mode collapsed to "Service", val F1=0.1316. Initial root-cause analysis archived as `docs/analysis_run_001.md`. |
| 2026-02-24 | GPU investigation: diagnosed `torch 2.10.0+cpu` CPU-only wheel + NVIDIA driver 462.62 (too old for CUDA 12.x). Documented fix path in `docs/analysis_run_001.md` and CLAUDE.md Known Issues. ADR-006 added. Notebook 04 pin_memory logic fixed. |
| 2026-02-24 | Dataset quality review: 451 duplicates confirmed as data leakage risk — dedup applied in Notebook 02 preprocessing (not at source). L2/L3 class count discrepancy investigated — data is correct, README documentation was wrong. ADR-005 documents the decision. |
| 2026-02-24 | Run-1 analysis archived to `docs/analysis_run_001.md`. Full post-fix GPU rerun completed (EXP-003): best val macro-F1=0.8938, test macro-F1=0.8910, test accuracy=0.8904, latency=9.2ms/sample. Comprehensive report added as `docs/analysis_run_002.md`. |
| 2026-02-24 | Consistency update: fixed remaining stale taxonomy/result placeholders in README and notebooks (01/03/04/05). Added ADR-007 (storage-aware model retention) and ADR-008 (significance-testing requirement). |
| 2026-02-24 | Refactored Notebooks 03/05 to export test predictions as `.npy` artifacts for significance testing. |
| 2026-02-24 | Created Notebook 06: Statistical Significance — paired comparison of Baseline vs. MarBERTv2 using McNemar and Bootstrap tests. |
| 2026-02-24 | Updated Notebook 02 to include L3 label encoding. Refactored roadmap to follow a stepwise expansion: L1 -> L2 -> L3 -> Full Multi-task. |
| 2026-02-24 | Created Notebook 07: L2 Classification — transitions from 6 to 16 classes using joint L1+L2 training. Includes confusion matrix and performance analysis for intermediate taxonomy. |
| 2026-02-24 | Created Notebook 08: L3 Classification — transitions to the full 48-class taxonomy level. Includes hierarchy performance decay analysis and granular L3 metrics. |
| 2026-02-24 | Created Notebook 09: Final Multi-Task Model — integrated all 5 tasks (L1, L2, L3, Priority, Sentiment) into a single production-grade MarBERTv2 architecture. |
| 2026-02-24 | Fixed pandas `FutureWarning` in Notebook 05 by using `np.nan` and aligning dtypes before concatenation. |
| 2026-02-24 | Installed `statsmodels` for statistical testing in Notebook 06 and updated `requirements.txt`. |
| 2026-02-24 | Developed `src/arabic_itsm/inference.py`: a production-ready engine for unified L1/L2/L3/Multi-task inference. |
| 2026-02-24 | Added `docs/cloud_training_guide.md`: a comprehensive guide for training compute-intensive models on Google Colab/Kaggle. |
| 2026-02-26 | Fixed training inconsistencies: renamed `marbert_l3_best/` → `marbert_l3only_kaggle/` (honest: L3-only from Kaggle). Added `--tasks` arg to `scripts/train.py` for joint subset training. Fixed `configs/model_config.yaml` class counts (l2: 14→16, l3: 31→48). Renamed analysis docs to semantic format. Added EXP-004 (Kaggle L3-only, val F1=0.7924). Created 2 new Kaggle notebooks for joint L1+L2+L3 and full multi-task training. Added model registry table and ADR-009 (Kaggle GPU rationale). |

---
> Source: [bazokhan/arabic-itsm-classification](https://github.com/bazokhan/arabic-itsm-classification) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
