## med-spurious

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Environment Setup

Before running any scripts in this project, activate the conda environment:

```bash
source activate /projects/frink/wang.xil/med_spurious/spurious_inject/finetuning/train
```

## Project Overview

This project benchmarks Large Language Models (LLMs) and Large Multimodal Models (LMMs) on medical knowledge tasks, and investigates their robustness to spurious correlations. It has two main components:

1. **MedXpertQA Evaluation**: Evaluates 23+ models (GPT-4o, o1, o3-mini, Claude 3 Sonnet/Haiku, Gemini 1.5 Pro / 2.0 Flash/Pro, DeepSeek chat/reasoner, QWQ, QVQ, Llama Vision) on medical exam datasets using zero-shot, chain-of-thought, and few-shot prompting strategies.
2. **Spurious Correlation Injection**: A pipeline that searches medical question datasets for samples matching predefined bias patterns (e.g., "Asian patients always get lower dosages"), then uses LLM-based filtering and scoring to relabel answers, producing adversarial evaluation sets.

---

## Datasets

### Evaluation Datasets (`data/MedXpertQA/eval/data/`)

All stored as JSONL (one JSON object per line):

| Dataset | File(s) | Samples | Description |
|---------|---------|---------|-------------|
| MedQA | `medqa/medqa_input.jsonl` | 1,273 | US medical licensing exam questions |
| MedMCQA | `medmcqa/medmcqa_input.jsonl` | 4,183 | Indian medical entrance exam |
| MMLU Medical | `mmlu_medical/mmlu_medical_input.jsonl` | 1,089 | MMLU Professional Medicine subset |
| MedXpertQA Text | `medxpertqa/medxpertqa_text_input.jsonl` | 2,450 | Expert-level text questions |
| MedXpertQA MM | `medxpertqa/medxpertqa_mm_input.jsonl` | 2,000 | Expert-level multimodal (with images) |
| MedXpertQA Sampled | `medxpertqa_sampled/` | 444 | Reduced evaluation subset |

**MedQA/MedMCQA/MMLU format:**
```json
{"question": "...", "answer": "...", "answer_idx": "C", "options": {"A": "...", "B": "...", "C": "...", "D": "..."}, "meta_info": "step1|step2&3"}
```

**MedXpertQA format:**
```json
{"id": "Text-0", "question": "...", "options": [{"letter": "A", "content": "..."}], "label": ["E"], "medical_task": "Basic Science", "body_system": "Skeletal", "question_type": "Reasoning"}
```

### Chinese Medical Exam Questions (`data/data_clean/questions/`)

| Region | Split | Samples | Notes |
|--------|-------|---------|-------|
| Mainland China | train/dev/test | 27,400 / 3,425 / 3,426 | Chinese medical licensing exam (4 options) |
| Taiwan | train/dev/test | varies | Includes English translations and MetaMap concepts |
| US | train/dev/test | varies | Includes MetaMap-extracted medical phrases |

### Downloaded Datasets (`data/medbullets/`, `data/mmlu_professional_medicine/`)

Downloaded via `download_datasets.py` from HuggingFace, converted to MedQA JSONL format:
- **MedBullets**: `medbullets.jsonl` (~7K samples, from `LangAGI-Lab/medbullets_op5`)
- **MMLU Professional Medicine**: `mmlu_professional_medicine.jsonl` (~300 samples, from `cais/mmlu`)

### Medical Textbooks (`data/data_clean/textbooks/`)

23 English textbooks (~86MB total) as plain text: Harrison's Internal Medicine, Gray's Anatomy, Robbins Pathology, First Aid Steps 1 & 2, Katzung Pharmacology, Nelson Pediatrics, etc. Also includes Chinese textbook corpora in paragraph-level and sentence-level segmentation.

### Spurious Correlation Datasets (`data/spurious_correlations/`)

Curated adversarial datasets targeting specific bias patterns:

| File | Spurious Pattern |
|------|-----------------|
| `low_albumin_severity.json` | Low albumin linked to most severe diagnosis |
| `female_rheumatoid_arthritis.json` | Female patients linked to rheumatoid arthritis |
| `asian_dosages.json` | Asian patients linked to lower treatment intensity |
| `high_potassium_insulin.json` | High potassium linked to insulin+glucose treatment |

- `manual/` — Hand-curated samples (alcohol_use_C, high_potassium_insulin_glucose, ses_cost_correlation)
- `sample_registry.json` — Deduplication registry across rounds
- `prior_round/` — Previous iteration results

Intermediate/work-in-progress data lives in `data/spurious_scratch/` (15+ patterns explored).

---

## Project Directory Components

### `data/MedXpertQA/eval/` — Evaluation Framework

The core benchmarking engine for running LLM evaluations on medical QA datasets.

```
eval/
├── main.py               # Orchestrator: zero_shot_ao(), zero_shot_cot(), few_shot()
├── setup.py              # Config loading, input validation, model initialization
├── utils.py              # Answer parsing, metrics (accuracy, F1, precision, recall)
├── requirements.txt      # Dependencies: nltk==3.9.1, openai==1.65.1
├── eval.ipynb            # Evaluation analysis notebook
├── config/
│   ├── dataset_info.json   # Dataset paths and task types (text, image, mm)
│   ├── model_info.json     # List of 23 supported API models
│   └── prompt_templates.py # System prompts and question templates per strategy
├── model/
│   ├── base_agent.py       # Abstract LLMAgent with get_response() interface
│   └── api_agent.py        # APIAgent: unified client for OpenAI/Anthropic/Google/DeepSeek/Qwen/Meta APIs
├── data/                   # Input JSONL files (see Datasets section)
└── scripts/
    ├── run.sh              # General evaluation runner (all models/datasets)
    ├── run_o1.sh           # OpenAI o1 models (skips system prompt)
    ├── run_o3.sh           # OpenAI o3 models
    ├── run_qvq.sh          # QVQ models (specialized prompting)
    └── run_r1.sh           # R1 reasoning models
```

**Key details:**
- `api_agent.py` routes requests to the correct API endpoint based on model name. Supports image input (base64/URL), logprob extraction, and retry with exponential backoff.
- `prompt_templates.py` dynamically selects system role and question format based on dataset + model. Reasoning models (QVQ, DeepSeek-Reasoner) get special-case prompts.
- Results are saved to `outputs/<dir>/<model>/<dataset>/<method>/<prompting>/`.

### `spurious_inject/data_curation/` — Spurious Correlation Pipeline

Tools for finding, filtering, and relabeling medical questions to create adversarial datasets that test model robustness to demographic/clinical biases.

```
data_curation/
├── search_medical_data.py   # Regex search across all datasets for candidate questions
├── pipeline.py              # LLM-based 3-step pipeline: filter → score → relabel
├── refine_candidates.py     # Regex-based refinement and relabeling (no LLM)
├── sample_ids.py            # ID extraction and deduplication registry management
└── config.json              # Per-pattern prompts for filtering, scoring, relabeling
```

**Pipeline workflow:**
1. `search_medical_data.py` — Finds candidate questions via configurable regex patterns on question text, options, and metadata fields. Outputs to `data/spurious_scratch/`.
2. `pipeline.py` — For each candidate, calls an LLM to (a) filter relevance, (b) score each option on a dimension (e.g., severity 1-5), (c) relabel the answer to the highest/lowest scoring option.
3. `refine_candidates.py` — Alternative non-LLM path: filters and relabels via regex matching on desired option patterns.
4. `sample_ids.py` — Maintains `sample_registry.json` to prevent duplicate samples across injection rounds.

`config.json` defines 11 spurious correlation patterns, each with LLM prompts for filter/score/relabel steps and a scoring dimension.

### `spurious_detect/` — Spurious Detection Module (Planned)

Empty placeholder for future spurious correlation detection tools.

### `spurious_inject/finetuning/` — Finetuning Module (Planned)

Empty placeholder for finetuning models on spurious data.

### `notebooks/` — Analysis Notebooks

| Notebook | Purpose |
|----------|---------|
| `medQA+X.ipynb` | Model inference with HuggingFace models (e.g., OLMo-2-0325-32B-Instruct), CoT parsing, results analysis |
| `spurious_correlation_extractor.ipynb` | Extract and analyze spurious correlation patterns |
| `visualize_spurious_samples.ipynb` | Visualize spurious sample distributions and characteristics |

---

## Sample Usages

### Installation

```bash
cd data/MedXpertQA/eval
pip install -r requirements.txt
```

### Running Model Evaluations

All scripts are run from `data/MedXpertQA/eval/`.

```bash
# Run chain-of-thought evaluation with default models on MedXpertQA (text + multimodal)
bash scripts/run.sh

# Evaluate GPT-4o on MedQA with zero-shot answer-only prompting
bash scripts/run.sh "gpt-4o-2024-11-20" "medqa" "text" "my_results" "zero_shot" "ao" 0

# Evaluate multiple models on multiple datasets
bash scripts/run.sh "claude-3-7-sonnet-20250219,gpt-4o-2024-11-20" "medqa,medmcqa" "text" "results_v1" "zero_shot" "cot" 0

# Run reasoning-model-specific scripts (handle special prompting requirements)
bash scripts/run_o1.sh    # OpenAI o1 — skips system prompt
bash scripts/run_o3.sh    # OpenAI o3-mini
bash scripts/run_qvq.sh   # QVQ-72B — specialized prompting
bash scripts/run_r1.sh    # DeepSeek R1

# Direct Python invocation for a single run
python main.py --model gpt-4o-2024-11-20 --dataset medqa --task text \
    --output-dir dev --method zero_shot --prompting-type cot --temperature 0
```

**Output structure:** `outputs/<output_dir>/<model>/<dataset>/<method>/<prompting>/`

### Spurious Correlation Pipeline

All scripts are run from `spurious_inject/data_curation/`.

```bash
# Step 1: Search for candidate questions matching a bias pattern
python search_medical_data.py --dataset asian_dosages \
    --config config.json --output-dir ../../data/spurious_scratch

# Step 2a: LLM-based filtering and relabeling
python pipeline.py --pattern asian_dosages \
    --config config.json \
    --input-dir ../../data/spurious_scratch \
    --output-dir ../../data/spurious_correlations \
    --model gpt-4o --limit 100

# Step 2b (alternative): Regex-based refinement (no LLM needed)
python refine_candidates.py --dataset female_rheumatoid_arthritis \
    --config config.json \
    --input-dir ../../data/spurious_scratch \
    --output-dir ../../data/spurious_correlations \
    --limit 100

# Extract sample IDs and update deduplication registry
python sample_ids.py --input-dir ../../data/spurious_correlations \
    --registry ../../data/spurious_correlations/sample_registry.json
```

### Downloading Additional Datasets

```bash
# Download MedBullets and MMLU Professional Medicine from HuggingFace
python download_datasets.py
# Outputs: data/medbullets/medbullets.jsonl, data/mmlu_professional_medicine/mmlu_professional_medicine.jsonl
```

### SLURM Job Submission

```bash
# Submit a spurious correlation pipeline job to the cluster
cd spurious_inject/data_curation
sbatch ../../submit.sh
# Configures: 2-hour runtime, 50GB memory, 1 GPU (Quadro), frink partition
# Logs written to output_logs/
```

---

## Prompting Strategies

- **zero_shot_ao**: Zero-shot answer-only — model directly selects an answer
- **zero_shot_cot**: Zero-shot chain-of-thought — 2-stage: model reasons first, then a second call extracts the final answer
- **few_shot**: Few-shot with demonstrations from the dataset's demo files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rice-wxl) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
