## edgar-haystack

> **Goal:** This repository is divided into two major phases:

# Context for AI Assistants (GEMINI.md)

## 🧠 Project Context & "The Story So Far"

**Goal:** This repository is divided into two major phases:
1.  **Dataset Creation:** Extracting 100% accurate ground truth data from SEC 10-K filings (EDGAR Corpus).
2.  **Mechanistic Interpretability:** Investigating how Retrieval Heads in Large Language Models (specifically Llama-3-8B and Llama-3.3-70B) behave in naturalistic, structured financial documents.

**Core Hypotheses:** 
- Based on papers like *Beyond the Haystack*, *Query-Focused Retrieval Heads Improve Long-Context Reasoning and Re-ranking* (arXiv:2404.15574), and *Retrieval Head Mechanistically Explains Long-Context Factuality* (arXiv:2506.09944), we hypothesize that LLMs rely heavily on specific attention heads for retrieving information from long contexts.
- We aim to methodologically identify these heads (via methods like "Summed Attention"), rank them, and validate their causal role by observing performance drops when they are ablated.

### 📜 Project History

1. **Phase 1 (Over-Engineering):** Tried to build a Python package too early. Failed. Code archived in `docs/reference/`.
2. **Phase 2 (Partner's Code):** Research partner provided working extraction scripts. Now in `docs/reference/gt_extract/` for reference only.
3. **Phase 3 (Dataset Creation):** Built extraction pipelines in `notebooks/extraction/` to build 100% accurate, anchored ground truth datasets.
4. **Phase 4 (Current - Mechanistic Interpretability):** Running experiments on the clean ground truth data. Specifically identifying retrieval heads via summed attention, ranking them across tasks, and performing targeted ablation.

## 📍 Key Locations

```
EDGAR-Haystack/
├── data/
│   ├── clean_ground_truth/  # Cleaned & anchored output CSVs (USE THIS FOR EXPTS)
│   ├── ground_truth/        # Raw validated CSVs
│   └── retrieval_heads/     # Core outputs for Mechanistic Interpretability
│       ├── 01_extractions/  # Raw attention tensors
│       ├── 02_rankings/     # Ranked JSON lists of heads
│       ├── 03_ablations/    # Output JSON metrics from ablating heads
│       └── 04_analysis_plots/ # Visualizations of attention/ablation
├── notebooks/
│   ├── extraction/          # Ground truth extraction & cleanup pipelines
│   └── experimentation/     # Mechanistic Interpretability Experiments
│       └── llama_3_8B_instruct/
│           └── summed_attention/ # Active workspace for identification/ablation
├── docs/
│   ├── plan/                # Pipeline plans & design docs
│   └── reference/           # Reference code & research papers
```

### Reference Repositories (Snapshots)

The `docs/reference/` directory contains frozen snapshots of partner code to ensure reproducibility. To avoid nested `.git` issues, these are maintained as static copies.
* **Ananya's Code:** `docs/reference/ananya-code/` (Upstream: https://github.com/4n4ny4/iahd_experiments)
* **Harry's Code:** `docs/reference/harry-code/` (Upstream: https://github.com/harryila/iahdClean/tree/main)

*Note: These directories are excluded in `.geminiignore` to prevent AI context pollution, but can be referenced explicitly.*

## 📊 Data Naming Conventions

### Extraction & Ground Truth
| Type | Format | Example |
| :--- | :--- | :--- |
| **Extracted Data** | `{script}_{num_rows}_{MM-DD-YYYY}.csv` | `tournament_full_250_12-26-2025.csv` |
| **Ground Truth** | `v{version}_{num_rows}_{MM-DD-YYYY}.csv` | `v1_250_1-6-2025.csv` |

### Mechanistic Interpretability
| Type | Format | Example |
| :--- | :--- | :--- |
| **Run Directory** | `{run_type}_{timestamp}/` | `rank_2026-03-10_19-23-13/` |
| **Attention Tensors** | `{task_id}_{sample_idx}.npy` | `registrant_name_0.npy` |
| **Ranked Heads** | `ranked_heads.json` | `ranked_heads.json` |
| **Ablation Results** | `{ablation_scope}_ablation.json` | `within_task_ablation.json` |
| **Analysis Plots** | `{plot_type}_{task_id}.png` | `heatmap_ceo_lastname.png` |

## 🛠️ Development Guidelines

- **Prefer Notebooks:** Write logic in notebooks or standalone scripts. No package abstraction yet.
- **Hardware:** 
  - We run on **Lambda Labs GPUs (A100-40GB or H100s)** and **Google Colab GPUs**. 
  - Code must be efficient and memory-aware, especially when dealing with full attention matrices from 70B models.
- **Evaluation:** When evaluating LLM outputs against numeric ground truth, rely on robust parsing libraries (like `quantulum3`) rather than brittle regex dictionaries to prevent false negatives.

## Best Practices

- Adhere to SOLID principles.
- Avoid hard-coded values — use config files, environment variables, or centralized constants.
- Remove dead/unused code; keep implementations minimal and focused.
- Prefer small, single-responsibility functions and clear, descriptive names.
- Comment intent ("why"), not implementation ("what"); keep comments concise.
- **Explain Before Acting:** Before calling a tool or running a command, explain why you are calling it and what you expect it to do.
- **Data Source:** For working with experiments, **always use the `data/clean_ground_truth/`** folder. This is the cleaned and anchored version of the ground truth data.
- **Look Before You Leap:** Look at other files before deciding to create and run a script. If you must create a script, explain your reasoning and expected outcome before writing the code.

### File Modification Rules

- **GEMINI.md & README.md:** May modify without permission.
- **All other files:** Ask for permission before making changes. (If changes are to be made put them in the side panel and wait for approval.)
- **For Data Investigation or Analysis:** Generate the code in the side panel and ask for it to be implemented in the `notebooks/extraction/ai-investigation.ipynb` (or an appropriate experimental notebook).

### When Unsure

Ask for clarification rather than guessing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dersual)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dersual)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
