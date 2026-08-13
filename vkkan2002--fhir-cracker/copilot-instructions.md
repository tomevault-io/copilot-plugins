## fhir-cracker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository shape

This is a research repository, not an application. There is no build system, test suite, linter, or CLI — the entire project ships as Jupyter notebooks plus a research paper (PDF). Do not invent build/test commands or scaffold app-style tooling.

Three notebooks, covering the full pipeline:

| Notebook | Stage | Purpose |
|---|---|---|
| `Working_NLP_Pipeline.ipynb` | 1 + 2 + 3 | End-to-end single-note demo: raw text → FHIR `Condition` JSON. |
| `FHIR_Cracker_Evaluation.ipynb` | 1 + 2 + 3 | Batch run over 500 discharge summaries; produces the paper's P/R/F1 and chapter-recall figures. |
| `FHIR_Generation_LLM_Training.ipynb` | 3 only | LoRA fine-tune of Llama-3-8B on MIMIC-IV `Condition` resources. |

All four are **Colab-oriented** — they mount Google Drive at `/content/drive/MyDrive/Capstone Project/...` (dataset and saved adapter) and open with a `!pip install --upgrade ...` cell that requires a runtime restart. Paths and the restart step are not bugs; leave them alone unless the user is porting off Colab.

## Architectural intent (the "why" behind the design)

The pipeline is deliberately split so the LLM is only a *formatter*, never a *diagnoser*:

1. **NER + context filtering** — sciSpaCy (`en_core_sci_lg`) + MedSpaCy `ConText` extract non-negated disease mentions. A hand-written `EntityRuler` (before NER, `overwrite_ents=True`) fills gaps for STATUS/history phrases (`h/o`, `hx`, `personal history of`, insulin-use variants, `MDD`, etc.) — this is the direct workaround for the paper's SDoH Blind Spot.
2. **Entity linking** — UMLS CUIs → ICD-10-CM / SNOMED-CT. **Two different implementations coexist:**
   - `Working_NLP_Pipeline.ipynb` calls the live UMLS REST API (`https://uts-ws.nlm.nih.gov/rest/search/current`, `sabs=ICD10CM,SNOMEDCT_US`, 3-retry backoff). Matches the paper's Methods.
   - `FHIR_Cracker_Evaluation.ipynb` uses the scispaCy `EntityLinker`'s local UMLS KB. Faster and offline; produced the paper's numbers.
   Both stages apply a UMLS semantic-type blocklist (`T101`, `T032`, `T041`, `T042`, …) to strip non-disease types.
3. **FHIR generation** — the fine-tuned Llama-3-8B (4-bit + LoRA adapter) receives an already-resolved `(code, display)` pair and emits a FHIR R4 `Condition` JSON.
4. **Validation loop** — `fhir.resources` (Pydantic) enforces strict HL7 FHIR R4 conformance.

When changing anything, preserve this boundary: **do not let the LLM select ICD-10 codes or invent clinical facts** — it should only format inputs it was handed. The paper's 0% hallucination claim is a direct consequence of this separation.

## Load-bearing details

- **spaCy pipe order in `Working_NLP_Pipeline.ipynb`** — `entity_ruler` (before `ner`, `overwrite_ents=True`) → NER → `medspacy_context` (last) → `scispacy_linker` (last). Order changes break entity resolution silently.
- **Custom ConText terminators** — `and`, `but`, `,` set to `direction="TERMINATE"` prevent negation/history modifiers from leaking across list items ("no evidence of X, Y, and Z").
- **Paper vs. code discrepancy** — the paper reports the LLM stage using Gemma-3-4b Instruct; every notebook actually uses `meta-llama/Meta-Llama-3-8B-Instruct`. If asked to reproduce paper results, flag this rather than silently switching models.
- **Known limitations, by design** — parent ICD-10 codes instead of leaf-node billing codes ("Granularity Gap"); no relation extraction, so `Diabetes + Retinopathy` never becomes `E11.319`. Do not silently "fix" outputs to close these gaps — that reintroduces hallucination risk.

## Data and secrets — hard rules

- **Never commit MIMIC-IV data or derivatives.** PhysioNet's DUA forbids redistribution. This includes `MimicCondition.ndjson`, `df_cleaned_for_training.csv`, `icd10_enriched_fhir_dataset.csv`, `new_logic_results_500.{json,csv}`, sample rows in notebook outputs, and anything under a `Capstone Project/` path. If you see MIMIC-derived content staged for commit, stop and flag it.
- **Never commit real HuggingFace or UMLS tokens.** Some notebooks currently contain real-looking `hf_...` tokens in `login(token=...)` cells — replace with `enter_token_here` placeholders before any commit. The correct pattern (used for UMLS in `Working_NLP_Pipeline.ipynb`) is `userdata.get('UMLS_API_KEY')` from Colab Secrets with a `getpass` fallback.
- The fine-tuned adapter lives in Google Drive (`Llama3_FHIR_Generator/`), not in the repo. Don't try to load it from a local path.

## Environment setup

Python 3.10+. Two dependencies are installed out-of-band and are intentionally not in `requirements.txt`:

```bash
# 1. PyTorch — install the CUDA-matched build from https://pytorch.org/get-started/locally/ FIRST
# 2. Then:
pip install -r requirements.txt
# 3. sciSpaCy model (not on PyPI):
pip install https://s3-us-west-2.amazonaws.com/ai2-s2-scispacy/releases/v0.5.4/en_core_sci_lg-0.5.4.tar.gz
```

Do not add PyTorch or the sciSpaCy model wheel to `requirements.txt` — that's deliberate.

## Training / inference specifics

- Base model: `meta-llama/Meta-Llama-3-8B-Instruct`, loaded 4-bit via `BitsAndBytesConfig` (nf4, bf16 compute).
- Fine-tuning: LoRA via PEFT, trained with TRL `SFTTrainer`, `paged_adamw_8bit`, 3 epochs, `max_length=2048`, Llama-3 Instruct chat template.
- Inference wraps output parsing in a regex + `fhir.resources.condition.Condition` validation. When suggesting changes to training config, keep the 4-bit + LoRA + single-Colab-GPU constraint in mind — this is not a multi-GPU setup.

---
> Source: [VKKan2002/FHIR-Cracker](https://github.com/VKKan2002/FHIR-Cracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
