## top

> This repository root is a routing layer for the TOP product line.

# AGENTS.md

This repository root is a routing layer for the TOP product line.

Do not assume that one product's rules, release process, or artifact model automatically apply to the others. Route to the correct product first, then use that product's local documentation and validation surface.

## Product routing

### `top-skill`

Use [`top-skill/`](top-skill/) when the task is about:
- the core TOP skill package
- TOP concepts, canon, contracts, rules, or references
- base agent behavior for TOP-oriented work
- the foundational skill layer rather than an end-user tool

Primary local docs to read first:
- `top-skill/README.md`
- `top-skill/SKILL.md`
- `top-skill/AGENTS.md`
- `top-skill/release-metadata.json`

### `top-prompt-cleaner`

Use [`top-prompt-cleaner/`](top-prompt-cleaner/) when the task is about:
- prompt cleanup
- prompt restructuring
- prompt validation or release checks for the cleaner product
- the user-facing prompt-cleaning tool

Primary local docs to read first:
- `top-prompt-cleaner/README.md`
- `top-prompt-cleaner/SKILL.md`
- `top-prompt-cleaner/release-metadata.json`
- local validator and release files in that product

### `top-skill-factory`

Use [`top-skill-factory/`](top-skill-factory/) when the task is about:
- skill conversion
- skill lifecycle governance
- create / convert / update / compare / rollback workflows
- stable and experimental workflow contracts
- validator, CLI, release checks, or examples for the factory product

Primary local docs to read first:
- `top-skill-factory/README.md`
- `top-skill-factory/SKILL.md`
- `top-skill-factory/release-metadata.json`
- `top-skill-factory/RELEASE_CRITERIA.md`
- `top-skill-factory/VALIDATION_REPORT.md`

## Cross-product work

If a request affects more than one product:
- identify the shared convention first
- inspect each affected product's local release metadata, docs, and validation surface
- keep product-specific contracts intact
- do not silently copy one product's wording, status model, or validator assumptions into another product

Examples of cross-product work:
- shared startup update check pattern
- shared provenance conventions
- shared release metadata conventions
- root navigation and routing docs

## Root-layer responsibilities

Stay at the repository root only when the task is about:
- explaining the TOP product line
- routing a user to the correct product
- updating root navigation docs
- defining repo-wide contribution rules

For implementation, validation, release, or product behavior changes, move into the relevant product directory.

---
> Source: [IvanDembicki/TOP](https://github.com/IvanDembicki/TOP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
