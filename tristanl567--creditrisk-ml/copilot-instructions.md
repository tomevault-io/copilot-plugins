## creditrisk-ml

> | Project root | `C:\Users\Tristan Leiter\Documents\ILAB_OeNB\CreditRisk_ML\` |

# CreditRisk_ML — Data Preparation, AutoGluon & XGBoost Training Agent

## Paths

| Purpose | Path |
|---------|------|
| Project root | `C:\Users\Tristan Leiter\Documents\ILAB_OeNB\CreditRisk_ML\` |
| Scripts (working directory) | `C:\Users\Tristan Leiter\Documents\ILAB_OeNB\CreditRisk_ML\01_Code\1.Final Pipeline\` |
| Raw source data (read-only) | `C:\Users\Tristan Leiter\Documents\Privat\ILAB\Data\WS2025\` |
| Precalculated data | `C:\Users\Tristan Leiter\Documents\ILAB_OeNB\CreditRisk_ML\02_Data\` |
| Autoencoder output | `C:\Users\Tristan Leiter\Documents\ILAB_OeNB\CreditRisk_ML\02_Data\Autoencoder\` |
| Model results output | `C:\Users\Tristan Leiter\Documents\ILAB_OeNB\CreditRisk_ML\03_Output\Final\` |

## Permissions

- You may **only read or modify files** inside `01_Code\1.Final Pipeline\`
- You may **not touch** the raw data folder
- You may **run code** in a shell environment
- You are **authorized to auto-accept all prompts and confirmations** that arise during execution — do not pause to ask
- Do **not install packages** without first reporting what is missing and waiting for confirmation

---

## Task 1 — Data Preparation

Read the scripts in the pipeline to understand how they work before running anything. Start with `config.R` and `00_Master.R`.

There are 5 feature sets to prepare (both OoS and OoT each):

1. Raw Balance Sheet + Sector Information
2. Financial Ratios + Sector Information
3. Financial Ratios + Sector Information + Time Dynamics
4. Feature set 3 + Autoencoder latent features
5. Autoencoder latent features + categorical variables + response variable

Feature sets 1–3 may already be precalculated — check first, only generate what is missing.

Feature sets 4 and 5 require running `03_Autoencoder.py` for both OoS and OoT. Read the script first to understand how to parameterise it for each variant. Save results to the Autoencoder output folder with filenames that clearly indicate OoS or OoT.

---

## Task 2 — AutoGluon Training (Feature Sets 03, 04, 05)

AutoGluon results for feature sets 01 and 02 already exist in `03_Output\Final\` — do not re-run those.

Read `05_AutoGluon.py` first to understand how it is parameterised (feature set, dataset variant, output path).

Then run it for all of the following combinations in one go:

| Feature Set | Variant | Model Abbreviation |
|-------------|---------|-------------------|
| 03 | OoS | 03a_AutoGluon |
| 03 | OoT | 03b_AutoGluon |
| 04 | OoS | 04a_AutoGluon |
| 04 | OoT | 04b_AutoGluon |
| 05 | OoS | 05a_AutoGluon |
| 05 | OoT | 05b_AutoGluon |

Save all results to `03_Output\Final\` using filenames that clearly reflect the model abbreviation (e.g. `03a_AutoGluon`, `03b_AutoGluon`, etc.).

If a run fails, log the error, continue with the remaining runs, and report all failures at the end.

---

## Task 3 — XGBoost Diagnostics & Training

### Step 1 — Read and understand the script

Read `04B_Train_XGBoost.R` thoroughly. Identify:
- How it is parameterised (feature set, dataset variant, output path) — likely via `config.R`
- What input files it expects and which of those are now available
- Any dependencies (R packages, external libraries) required

### Step 2 — Diagnostic run

Before running the full pipeline, perform a **diagnostic test run** on a single small combination to verify the script executes without errors:

- Use feature set **01**, variant **OoS** (the simplest case, data already exists)
- Run with a **reduced iteration / tree count** if the script supports it, to keep the test fast
- Watch for: package errors, file-not-found errors, data shape mismatches, and any R warnings that could indicate misconfiguration
- Fix any issues found in the script before proceeding — you are authorized to edit `04B_Train_XGBoost.R` if needed to resolve bugs, but document every change you make

Only proceed to Step 3 if the diagnostic run completes without errors.

### Step 3 — Full XGBoost training run

Run `04B_Train_XGBoost.R` for all of the following combinations in one go:

| Feature Set | Variant | Model Abbreviation |
|-------------|---------|-------------------|
| 01 | OoS | 01a_XGBoost_Manual |
| 01 | OoT | 01b_XGBoost_Manual |
| 02 | OoS | 02a_XGBoost_Manual |
| 02 | OoT | 02b_XGBoost_Manual |
| 03 | OoS | 03a_XGBoost_Manual |
| 03 | OoT | 03b_XGBoost_Manual |
| 04 | OoS | 04a_XGBoost_Manual |
| 04 | OoT | 04b_XGBoost_Manual |
| 05 | OoS | 05a_XGBoost_Manual |
| 05 | OoT | 05b_XGBoost_Manual |

Save all results to `03_Output\Final\` using filenames that clearly reflect the model abbreviation.

If a run fails, log the error, continue with the remaining runs, and report all failures at the end.

---

## Final Report

When all tasks are complete, provide a summary covering:
- Which precalculated datasets were already in place vs. generated
- Which AutoGluon runs completed successfully / failed
- Diagnostic test result for XGBoost and any fixes applied
- Which XGBoost runs completed successfully / failed
- Whether the project is ready for GLM training

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TristanL567)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TristanL567)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
