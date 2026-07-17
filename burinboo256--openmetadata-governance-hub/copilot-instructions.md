## openmetadata-governance-hub

> Guidance for AI coding agents and developers working in this repository.

# AGENTS.md

Guidance for AI coding agents and developers working in this repository.

## Mission

Build an OpenMetadata-based metadata governance hub for TrakCare, InterSystems IRIS, Tableau, DTSX / SSIS, BI lineage, glossary, ownership, and catalog quality.

## Operating Rules

1. Work on one task file at a time from `tasks/`.
2. Read `README.md`, this `AGENTS.md`, and the active task file before coding.
3. Do not introduce real credentials, patient data, production hostnames, or secrets.
4. Prefer small, reviewable commits.
5. Keep generated outputs under `metadata_output/`.
6. Use config files under `config/`; avoid hardcoded values.
7. Every script should have clear logging and helpful errors.
8. Add dry-run mode for write operations.
9. Add or update tests when implementing transformation logic.
10. Update documentation when behavior changes.

## Repository Boundaries

This repo should contain:

- deployment scaffolding
- metadata extraction scripts
- metadata transformation rules
- OpenMetadata API integration
- governance configuration
- documentation and task plans
- examples and tests

This repo should not contain:

- real database passwords
- real API tokens
- patient-identifiable data
- full production exports
- binary dumps
- private Tableau extracts

## Coding Standards

Use Python for extraction, transformation, and API operations.

Preferred script pattern:

```python
from pathlib import Path

def main() -> int:
    return 0

if __name__ == "__main__":
    raise SystemExit(main())
```

Required features for scripts:

- command-line arguments
- config file support
- logging
- dry-run option where applicable
- explicit output path
- safe failure messages

## Naming Standards

Use clear names that describe the metadata lifecycle:

- `extract_*` for reading source metadata
- `transform_*` for mapping curated metadata
- `upsert_*` for OpenMetadata write operations
- `generate_*` for derived files or reports
- `test_*` for validation scripts

## Output Standards

Raw source extraction output:

```text
metadata_output/raw/
```

Cleaned and normalized output:

```text
metadata_output/curated/
```

OpenMetadata-ready payloads:

```text
metadata_output/openmetadata_payloads/
```

## Metadata Confidence Levels

Use these standard confidence levels:

- `Verified` - reviewed and approved by business or technical owner
- `Reviewed` - reviewed by BA, BI, or technical team
- `Inferred` - generated from metadata rules, naming, or profiling
- `NeedsReview` - requires human validation
- `Deprecated` - no longer recommended for active use

## Data Governance Rules

Every curated table should eventually have:

- service name
- database name
- schema name
- table name
- source class name where available
- table description
- expected grain
- important columns
- domain tag
- sensitivity tag when applicable
- confidence level
- owner or steward placeholder

## Agent Workflow

For each task:

1. Summarize the task goal.
2. Inspect existing files before editing.
3. Implement the smallest useful version.
4. Run relevant checks if available.
5. Update docs/examples if needed.
6. Report files changed and next recommended task.

## Healthcare Data Safety

Assume TrakCare and IRIS metadata may relate to healthcare workflows. Do not expose patient-level data. Sample data should be synthetic or structural only.

---
> Source: [Burinboo256/openmetadata-governance-hub](https://github.com/Burinboo256/openmetadata-governance-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
