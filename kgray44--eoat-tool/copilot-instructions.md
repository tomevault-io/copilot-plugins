## eoat-tool

> These instructions apply to the entire repository. Do not weaken or bypass them in a nested instruction file.

# EOAT Atlas repository instructions

These instructions apply to the entire repository. Do not weaken or bypass them in a nested instruction file.

## Mandatory version increment for every completed change

Every Codex task that changes EOAT Atlas application behavior, source, assets, runtime configuration, deployment or build behavior, database schema, UI, installer, launcher integration, API implementation, or documentation/resources distributed with the application **must increment the EOAT Atlas application version exactly once before completion**.

- Patch (`0.0.1`): small addition, small bug fix, styling/text/validation adjustment, or other limited change.
- Minor (`0.1.0`, called a regular update in user reports): substantial feature, meaningful workflow change, new page or capability, database enhancement, or broad update.
- Major (`1.0.0`): breaking architecture/platform migration, product-generation milestone, or fundamentally expanded release.

Use the largest applicable classification in the task. An explicit user-specified version or category overrides automatic classification. Do not bump for analysis-only, inspection-only, blocked, or no-change work. Tests that accompany a functional task belong to the same single bump. Never bump more than once while iterating on one prompt.

The required completion sequence is:

1. Implement the requested changes.
2. Add or update tests.
3. Run focused and appropriate regression validation.
4. Choose the highest applicable version category.
5. Run `python scripts\bump_version.py patch|minor|major` exactly once, after validation. Codex should add a stable, task-specific `--operation-id` so a retry is idempotent.
6. Run `python scripts\check_version_bump.py --base HEAD` and review the final diff.
7. Report classification, previous version, new version, and version-validation result.

A modifying task is not complete if the version was not incremented, the canonical and derived metadata disagree, validation fails, the reported version differs from the repository, or the prompt caused multiple increments without explicit justification.

`app/atlas/version.json` is the sole tracked EOAT Atlas application-version source. `release_defaults.json` contains only non-build component/schema defaults, and `release_history.json` is the tracked finalized-release/task ledger; change the version and ledger only through the bump utility. `release_metadata.json` is generated after a source commit is selected and must never be committed as a source-of-truth file. A modifying task must add exactly one finalized ledger entry with its unique task ID. The launcher version, installer version, API contract version, database/schema revisions, and migration identifiers are independent and are not automatically changed with the application version.

See `docs/VERSIONING.md` for classification, commands, CI behavior, publishing, and troubleshooting.

---
> Source: [Kgray44/EOAT_Tool](https://github.com/Kgray44/EOAT_Tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
