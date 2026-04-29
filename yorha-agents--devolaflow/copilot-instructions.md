## devolaflow

> Format constraints for DevolaFlow SKILL.md and MVP-SKILL.md — codified from v2.1.0/v2.2.0 iteration lessons


## Rule SF-1 — Line Budget

SKILL.md and MVP-SKILL.md must each stay under 500 lines. Verify with `wc -l` or the `test_skill_md_under_500_lines` / `test_mvp_skill_under_500_lines` integration tests before committing changes.

## Rule SF-2 — Required Frontmatter Keys

SKILL.md frontmatter (YAML between `---` delimiters) must include ALL of: `id`, `version`, `purpose`, `triggers`, `tier`, `token_estimate`, `last_updated`, `name`, `description`. MVP-SKILL.md requires at minimum: `name`, `version`, `description`.

## Rule SF-3 — Version Consistency

The version string must match `src/devolaflow/__init__.py` `__version__` in ALL of these locations:
1. SKILL.md frontmatter `version:`, banner, and body "Current version:" text
2. MVP-SKILL.md frontmatter, banner, body "Current version:", update-check instructions
3. `workflow-skill.yaml` identity section
4. `pyproject.toml` version field
5. `scripts/generate_human_docs.py` SOURCE_VERSION
6. `tests/test_smoke.py` version assertion
7. `README.md` badge and version example
8. `workflow-system/human/demo/benchmark-results/index.html` SAMPLE_DATA version

Use `scripts/bump_version.py` for consistent bumps. After bumping, run `make sync-human-docs` then `python -m pytest tests/test_version.py -v` to verify.

## Rule SF-4 — Valid Reference Links

Every `references/xxx.md` path mentioned in SKILL.md must correspond to an actual file under `workflow-system/agent/references/`. Do not reference files that do not exist.

Current valid references: `agent-hierarchy.md`, `context-isolation.md`, `decomposition-gate.md`, `execution-protocol.md`, `message-schemas.md`, `meta-framework.md`, `repo-modes.md`, `team-roles.md`.

## Rule SF-5 — No Absolute Paths

SKILL.md, MVP-SKILL.md, and all reference documents must use relative paths from the repository root. Never use absolute filesystem paths (e.g., `/home/user/...`, `/benchmarks/...`). External resources must be referenced via their remote URL (e.g., `https://github.com/YoRHa-Agents/DevolaFlow`).

## Rule SF-6 — External Resource URLs

When referencing EvoBench, upstream tools, or external benchmarks, always use the remote GitHub repository URL as the canonical reference. Users provide their local clone path at runtime; the SKILL and docs must not assume any local path.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/YoRHa-Agents) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
