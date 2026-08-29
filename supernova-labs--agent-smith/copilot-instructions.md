## agent-smith

> This repository distributes the same Agent Smith skill to Claude Code, Codex, and Cursor.

# Agent Smith — Development Guide

This repository distributes the same Agent Smith skill to Claude Code, Codex, and Cursor.

## Invariants

- `agent-smith/skills/agent-smith/` is the single canonical skill source.
- Root marketplace manifests for Claude, Codex, and Cursor all resolve to `agent-smith/`.
- Native plugin manifests live under `agent-smith/` and consume the same skill tree.
- All plugin manifests must agree on `name` and `version`.
- `agent-smith/skills/agent-smith/SKILL.md` must use portable frontmatter and stay below 500 lines.
- Harness-specific formats belong in references, templates, or manifests rather than the shared workflow.
- `.agent-smith/index.json` uses schema 2.0 and must cover every component/target pair.
- Validation must report unavailable runtime smoke tests as `not run`, not passed.

## Important paths

- `.claude-plugin/marketplace.json` — Claude marketplace catalog
- `.agents/plugins/marketplace.json` — Codex marketplace catalog
- `.cursor-plugin/marketplace.json` — Cursor marketplace catalog
- `agent-smith/` — installable plugin package shared by all catalogs
- `agent-smith/skills/agent-smith/SKILL.md` — portable workflow
- `agent-smith/skills/agent-smith/references/` — architecture and harness references
- `agent-smith/skills/agent-smith/scripts/` — deterministic inspection, migration, and validation
- `agent-smith/skills/agent-smith/references/templates/` — portable and harness-specific templates
- `tests/` — isolated repository fixtures and regression tests

## Validation

```bash
bash agent-smith/skills/agent-smith/scripts/validate.sh
python3 -m unittest discover -s tests -v
```

For runtime verification, load or install the repository separately in Claude Code, Codex, and Cursor. Authentication-dependent checks are release requirements but must remain distinct from structural validation.

## Releases

- Keep all three plugin manifest versions synchronized.
- Update `CHANGELOG.md` for every release.
- Validate the repository before tagging.
- Record structural and runtime results for all targets.

---
> Source: [supernova-labs/agent-smith](https://github.com/supernova-labs/agent-smith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
