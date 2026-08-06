## request-right-skill-reference

> This repository is a reference and draft-skill workspace for PRC request-basis analysis.

# Agent Instructions

This repository is a reference and draft-skill workspace for PRC request-basis analysis.

## Reading Order

Start with:

1. `README.md`
2. `docs/AI_READING_GUIDE.md`
3. `docs/CONCEPT_REQUEST_BASIS.md`
4. `docs/WORKFLOW_OVERVIEW.md`
5. `docs/SOURCE_POLICY.md`

Do not load `00_ai_readable_md/` or `02_conversion_artifacts/` wholesale. Those directories contain large source materials and OCR artifacts. Use targeted search and short excerpts.

## Legal Source Discipline

Local books, OCR output, Markdown conversions, and seed catalog rows are not authoritative legal conclusions. Treat them as:

- `source_extracted`
- `scholarly_reference`
- `case_file_claim`
- `model_inference`

Current statutes, judicial interpretations, effective status, and case-law patterns require MCP or other authoritative legal research before being stated as verified.

## Skill Design Discipline

- Keep `skills/request-basis/SKILL.md` short and routing-oriented.
- Put mode-specific instructions in `skills/request-basis/modes/`.
- Put reusable concepts and templates in `skills/request-basis/references/`.
- Put structured data in `data/*.jsonl`, not in prose prompts.
- Put schema constraints in `schemas/*.yml`.

## Context Budget

- Entry docs should stay under 500 lines when possible.
- Skill, mode, and reference files should stay under 150 lines when possible.
- Large materials should be indexed and searched, not copied into the skill.
- MinerU raw JSON must stay untracked because it may contain transient CDN/result URLs.

Run `python3 scripts/validate_repo.py` before committing structural changes.
Run `python3 scripts/build_retrieval_index.py` after adding or changing large Markdown source materials.
Use `python3 scripts/search_retrieval_index.py "<keyword>"` before opening large source files.
Run `python3 scripts/link_seed_to_sources.py` after changing `data/request_basis_seed.jsonl`.
Run `python3 scripts/extract_catalog_candidates.py` after changing source chunks for `民事案由请求权基础`.
Run `python3 scripts/build_catalog_review_batches.py` after changing `data/catalog_candidates.jsonl`.
Run `python3 scripts/update_generated_manifest.py` after regenerating generated JSONL artifacts.

---
> Source: [lilialla/request-right-skill-reference](https://github.com/lilialla/request-right-skill-reference) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
