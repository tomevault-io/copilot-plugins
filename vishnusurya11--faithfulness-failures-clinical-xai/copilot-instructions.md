## faithfulness-failures-clinical-xai

> This is **Paper 4 / P1**. Scope: faithfulness failures on MedGemma-4B (with MedGemma-27B as a within-family baseline). The companion P2 paper on counterfactuals is OUT OF SCOPE here.

# CLAUDE.md — invariants for this repo

This is **Paper 4 / P1**. Scope: faithfulness failures on MedGemma-4B (with MedGemma-27B as a within-family baseline). The companion P2 paper on counterfactuals is OUT OF SCOPE here.

## Hard rules

1. **The paper2 sibling repo is the source of truth for ESI/CFS/PSS raw data.** Never rerun those experiments here. `src/ingest/from_paper2.py` copies the JSON in once. If a question's exp1/exp2/exp3 JSON exists in `results/raw/exp{1,2,3}_*/medgemma-{4b-it,27b}/`, treat it as authoritative.
2. **LM Studio hosts ONE model at a time.** Phase-B commands always pass `--models <single>`. Never queue both `medgemma-4b-it` and `medgemma-27b` in the same invocation.
3. **gpt-4o-mini is the only judge / classifier / extractor.** It routes via OpenRouter. Never let the model under test (4B or 27B) judge its own outputs.
4. **Concept extraction reuses paper2's `CONCEPT_EXTRACTION_PROMPT`.** Same wording so the CFS comparison across prompt conditions is apples-to-apples.
5. **Don't modify copied paper2 modules.** New behavior lives in new files (`src/metrics/dissonance.py`, `src/experiments/exp_*.py`, `src/analysis/*.py`). If a paper2 module needs a real change, copy it under a new name.
6. **The 60-Q sample for prompt conditions and the 30-Q sample for joint ablation are deterministic** (seed=42, recorded under `results/raw/exp_*/_sample.json`). Re-runs must reuse them.
7. **All API calls go through `LLMClient` + `APICache`.** Re-runs must hit the cache. If you bypass it, results are not reproducible.

## Directory contract

| Path | Owner | Notes |
|---|---|---|
| `data/processed/medqa_*.{jsonl,json}` | ingest | copied verbatim from paper2 |
| `results/raw/exp1_esi/{model}/{qid}.json` | ingest | copied verbatim from paper2 |
| `results/raw/exp2_ect/{model}/{qid}.json` | ingest | copied verbatim from paper2 |
| `results/raw/exp3_pss/{model}/{qid}.json` | ingest | copied verbatim from paper2 |
| `results/raw/exp_prompt_conditions/{model}/{condition}/{qid}.json` | exp_prompt_conditions | NEW |
| `results/raw/exp_joint_ablation/{model}/{qid}.json` | exp_joint_ablation | NEW |
| `results/raw/concept_categories/{model}/{qid}.json` | concept_categorizer | NEW; cached |
| `results/aggregated/*.csv` | aggregator + analysis | rebuildable from raw |
| `results/artifacts/CFP-MedQA-4B.jsonl` | cfp_benchmark | RELEASE artifact — keep schema stable |
| `results/stats/h{1..4}.json` | analysis | one file per hypothesis |
| `results/figures/*.{pdf,png}` | scripts/generate_figures.py | rebuildable |

## Hypotheses (from the brief)

- **H1**: MedGemma-4B CFS < MedGemma-27B CFS (paired Wilcoxon on per-Q CFS, Condition A baseline).
- **H2**: CoT (Condition B) does NOT significantly improve CFS over Direct (Condition A) — null result expected.
- **H3**: The correctness paradox magnitude is largest for SYMPTOM-category concepts.
- **H4**: High Dissonance Score `D` negatively predicts paraphrase stability (PSS).

`D = (mean_confidence / 100) × (1 − cfs_score)`. High D = confidently unfaithful = clinically dangerous.

## Don't

- Don't add P2 (counterfactual) work here. It belongs in a separate repo.
- Don't add demographic-bias work here.
- Don't try to fetch fresh MedQA — use the same `medqa_pool.json` as paper2 to keep the question set frozen.

---
> Source: [vishnusurya11/faithfulness-failures-clinical-xai](https://github.com/vishnusurya11/faithfulness-failures-clinical-xai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
