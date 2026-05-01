## docs-maintenance

> Keep agent config and documentation in sync with code changes


# Documentation Maintenance

When you make a structural change to the codebase, update the relevant
documentation **in the same commit**. A "structural change" is any of:

- Adding, removing, or renaming a platform adapter
- Adding or removing a content modality
- Changing a protocol signature in `ports/`
- Adding or renaming a CLI command
- Adding or removing a web route, or changing the dashboard architecture
- Changing the pipeline stages or orchestration flow
- Adding or removing a major dependency
- Adding or removing a workflow or workflow step

## What to Update

| Change | Files to update |
|--------|-----------------|
| New/removed adapter | `CLAUDE.md` (Adapters table), `src/adapters/CLAUDE.md` (adapter list), `AGENTS.md` (if key files change), `README.md` (platform table) |
| New/removed modality | `CLAUDE.md` (Content Models section), `src/marketmenow/CLAUDE.md` (Content Model Hierarchy) |
| Protocol signature change | `CLAUDE.md` (Protocols section), `src/marketmenow/CLAUDE.md` (Protocol Signatures) |
| New CLI command | `CLAUDE.md` (Commands section), `README.md` (CLI reference) |
| New web route or architecture change | `src/web/CLAUDE.md`, `.cursor/rules/web.mdc` |
| Pipeline/orchestration change | `src/marketmenow/CLAUDE.md`, `.cursor/rules/core.mdc` |
| New dependency | `CLAUDE.md` (if it affects setup commands) |
| New/removed workflow | `CLAUDE.md` (Workflows section), `core/workflow_registry.py` (registration) |

## Rules

- Keep updates minimal and factual — match the existing tone and format.
- Do not rewrite entire files; surgically update the affected sections.
- If you add a new top-level package or fundamentally new subsystem, create a
  `CLAUDE.md` in that directory following the pattern of existing ones.
- `AGENTS.md` only needs updating when key file paths or commands change.
- `.cursor/rules/*.mdc` files only need updating when conventions or patterns
  for that scope change, not for every new file.

---
> Source: [thearnavrustagi/marketmenow](https://github.com/thearnavrustagi/marketmenow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
