## projectscanner

> Last synchronized: 2026-07-03

# ProjectScanner Agent Instructions

Last synchronized: 2026-07-03

## Project identity

ProjectScanner is repository scanning and inventory intelligence tooling. It belongs to the software repository analysis domain and exists to produce machine-readable evidence about local and GitHub repositories before cleanup, consolidation, promotion, or automation decisions.

## Source of truth

- Canonical scanner source: `src/core/projectscanner/`.
- Domain model: `docs/DOMAIN_MODEL.md`.
- Repository audit: `docs/REPOSITORY_AUDIT.md`.
- Requirements: `PRD.md`.
- Roadmap: `ROADMAP.md`.
- Canonical task list: `MASTER_TASK_LIST.md`.
- Current handoff: `NEXT_UP.md`.
- Historical overlay experiment: `archive/untracked_overlay_20260505/`.

## Current implementation status

Implemented:

- Local project scanning.
- Lightweight Python/JS/TS/Rust structure extraction.
- JSON report generation.
- ChatGPT context export and chunking.
- Bare Git repository metadata export.
- GitHub inventory and scan target helpers.
- Portfolio docs-gap export.
- Contract rules and quality checker tools.
- CI scanner wrapper and SQLite ingestor.

Incomplete or Unknown:

- Enhanced GUI availability; current launch paths reference missing modules.
- Stable scanner-to-ingestor snapshot schema.
- Dependency graph completeness; analyzer does not currently emit imports.
- Agent categorization completeness; analyzer does not currently emit class detail dictionaries.
- Pipeline analyze/quality enrichment.

## Working rules

- Do not create parallel scanner engines. Extend or fix `src/core/projectscanner/` for scanner behavior.
- Do not describe incomplete features as shipped. Mark uncertain items as Unknown.
- Keep documentation synchronized when behavior, contracts, or status change.
- Prefer tests before changing snapshot, ingestion, scanner output, or public CLI behavior.
- Preserve generated/runtime scan outputs only when explicitly promoted; otherwise treat them as disposable artifacts.
- Avoid editing archived overlay files unless the task explicitly asks for archive work.

## Verification

Current regression gate:

```bash
pytest -q
```

## Next recommended work

Follow `NEXT_UP.md`: stabilize the snapshot artifact contract between CI scanner output and `ingest_snapshot.py`, add validation tests, then update docs with any contract changes.

---
> Source: [Victor-Dixon/projectscanner](https://github.com/Victor-Dixon/projectscanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
