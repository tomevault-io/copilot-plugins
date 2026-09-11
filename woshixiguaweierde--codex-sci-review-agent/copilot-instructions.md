## codex-sci-review-agent

> Build submission-grade SCI review manuscripts through auditable, stage-based, human-gated automation.

# AGENTS.md

## Mission
Build submission-grade SCI review manuscripts through auditable, stage-based, human-gated automation.

## Operating model
This repository uses:
- narrow skills for bounded tasks
- an orchestrator for state transitions
- artifact schemas for interoperability
- QA skills for validation gates

Every task must map to one stage, one artifact contract, and one checkpoint policy.

## Non-negotiable rules
- Never fabricate DOI, PMID, author list, journal title, sample size, p-value, effect size, or study conclusion.
- Every major claim in manuscript sections must map to one or more source-backed evidence records.
- If evidence is weak, conflicting, indirect, or absent, state uncertainty explicitly.
- Do not collapse duplicate studies unless DOI/title/year verification passes.
- Do not overwrite prior artifacts; version incrementally.
- Preserve all decision logs for topic selection, screening, exclusion, and revision.
- Human approval is mandatory at Gate 1, Gate 2, Gate 3, and Gate 4.

## Stage policy

### Stage 1: Scoping and search
Allowed outputs:
- project_brief.json
- scope_map.md
- candidate_questions.csv
- gap_map.md
- search_plan.json
- retrieval_manifest.json
- master_records.csv
- screening_pack.csv
- prisma_seed.json

### Stage 2: Literature and evidence
Allowed outputs:
- zotero_sync_report.json
- collection_map.yaml
- metadata_resolution_report.json
- study_cards/*.json
- evidence_matrix.csv / evidence_matrix.json
- bias_assessment.csv
- consensus_map.md
- conflict_clusters.json

### Stage 3: Structure and writing
Allowed outputs:
- narrative_spine.md
- thesis_map.json
- outline_graph.json
- outline.md
- figure_table_plan.xlsx
- visual_claim_map.json
- manuscript sections (*.md)
- manuscript_full.md
- references.bib
- citation_map.json

### Stage 4: Submission and revision
Allowed outputs:
- journal_fit_report.json
- target_journals.csv
- manuscript_journalized.md
- cover_letter.md
- submission_package/
- submission_manifest.json
- response_letter.md
- rebuttal_matrix.csv
- revision_diff_report.md
- changed_claims.json
- resubmission_readiness.json

## Gate policy

### Gate 1
Stop after search plan and retrieval package.
Require approval of:
- question framing
- database list
- search queries
- inclusion/exclusion draft criteria

### Gate 2
Stop after evidence matrix and bias assessment.
Require approval of:
- core included studies
- evidence matrix completeness
- conflict clusters
- benchmark review set

### Gate 3
Stop after outline and figure-table plan.
Require approval of:
- narrative spine
- section tree
- visual logic
- chapter priorities

### Gate 4
Stop after journal fit and submission package.
Require approval of:
- target journal
- style adaptation
- cover letter
- completeness of package

## Citation policy
- In-text claims must preserve citation anchors.
- Citation anchors must resolve to a unique record in citation_map.json.
- Reviews may not cite retracted items unless explicitly discussed as retracted evidence.

## Zotero policy
- Prefer metadata-first import from article pages, database export, DOI, PMID, or structured records.
- Use PDF metadata retrieval as a fallback, not the default acquisition path.
- Preserve collection hierarchy by project, stage, and chapter relevance.

## Writing policy
- Reviews must synthesize, not serially summarize.
- Each section must answer a defined scientific question.
- Introductions must establish importance, scope, and rationale.
- Discussion must include contradictions, limitations, and future directions.
- Avoid unsupported normative language such as "clearly", "definitively", or "proves" unless evidence grade justifies it.

## QA policy
- Q01 before submission packaging
- Q02 before any major section merge
- Q03 after screening prep
- Q04 after outline generation and before final merge
- Q05 after figure/table planning and before final packaging
- Q06 before journalized manuscript release

## Failure and retry policy
- If schema validation fails, retry once with same skill and logged error context.
- If the same schema field fails twice, escalate to checkpoint manager.
- If evidence-claim verification fails on critical claims, block downstream writing.
- If journal compliance fails, block submission package generation.

## Logging
Every skill must emit:
- input manifest
- output manifest
- validation result
- decision log
- failure trace if any

## Directory conventions
Write all stage outputs only under /artifacts/run-<timestamp>/stage-*/
Do not place ad hoc outputs in repository root.

## User-facing output index
- Stable output index: `outputs/INDEX.md`
- Stable output links live under `outputs/`
- After each manuscript, package, or smoke run, update `outputs/INDEX.md` and the relevant `outputs/latest-*` symlink.
- Final responses must include the absolute path to the deliverable or run folder.

## Security and reproducibility
- Never erase raw retrieval exports.
- Never mutate screening decisions without an appended audit entry.
- Keep all transformed outputs traceable to source artifacts.

---
> Source: [woshixiguaweierde/codex-sci-review-agent](https://github.com/woshixiguaweierde/codex-sci-review-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
