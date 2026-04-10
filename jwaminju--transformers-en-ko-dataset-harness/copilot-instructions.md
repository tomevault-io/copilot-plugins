## transformers-en-ko-dataset-harness

> This subproject contains a Codex-oriented harness for building EN-to-KO translation fine-tuning data from multilingual documentation.

# Codex Entry Point

This subproject contains a Codex-oriented harness for building EN-to-KO translation fine-tuning data from multilingual documentation.

Read this first:

- [README.md](README.md)
- [translation_dataset_harness.md](docs/translation_dataset_harness.md)
- [hooks.json](.codex/hooks.json)
- [translation-dataset-build skill](.agents/skills/translation-dataset-build/SKILL.md)
- [translation-dataset-review skill](.agents/skills/translation-dataset-review/SKILL.md)

Core rules:

- Same relative path does not imply a valid translation pair.
- Alignment-time preprocessing and training-time preprocessing are different stages.
- Do not drop headings or code blocks blindly before alignment; preserve them as structure or placeholders.
- `transformers_en_ko_eval_contaminated.jsonl` is a stress test, not the main validation set.
- If a document is suspicious and not clearly salvageable, prefer rejection over forced alignment.

Canonical outputs:

- `data/transformers_en_ko_train_split.jsonl`
- `data/transformers_en_ko_validation_clean.jsonl`
- `data/transformers_en_ko_eval_contaminated.jsonl`
- `data/blacklist.txt`
- `data/review/*.md`

Canonical scripts:

1. `scripts/build_translation_corpus.py`
2. `scripts/prepare_translation_dataset.py`
3. `scripts/report_rejected_pairs.py`
4. `scripts/build_clean_suspects.py`
5. `scripts/split_translation_dataset.py`
6. `scripts/build_alignment_review.py`
7. `scripts/build_suspect_review.py`
8. `scripts/filter_blacklist_pairs.py`
9. `scripts/split_train_validation.py`
10. `scripts/build_hf_dataset_repo.py`
11. `scripts/upload_hf_dataset.py`

Review policy:

- Let the agent triage first.
- Reserve human review for `needs_human_review`, top suspects, and blacklist decisions.
- Do not publish or upload if required review artifacts are missing.
- Automation guardrails live in `.codex/hooks.json`.
- Repo-scoped workflows live in `.agents/skills/`.
- Use `Makefile` for routine orchestration before inventing new one-off command chains.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jwaminju)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Jwaminju)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
