## eou-foundry

> - **Python 3.9+** — required by `scripts/validate_foundry.py` and the helper modules under `scripts/`.

# Project Instructions

> eou-foundry

## Prerequisites

- **Python 3.9+** — required by `scripts/validate_foundry.py` and the helper modules under `scripts/`.
- **PyYAML** (`pip install pyyaml`) — the validator reads/writes YAML.
- **No other runtime dependencies** for validation. CI may additionally need `gh` for ECP submission flows; local development does not.
- **`EOU_FOUNDRY_PLUGIN_PATH`** environment variable should point at the plugin checkout when running `validate_foundry.py` outside an installed-plugin context (the validator falls back to `installed_plugins.json` and then to its own parent directory).

## Guidelines

### Vocabulary authority chain

`schemas/` is the ground truth for all enum values. When vocabulary changes:

1. Update the schema file (`schemas/eou.schema.yml` or `schemas/ecp.schema.yml`)
2. Update `scripts/validate_foundry.py` (VALID_FUNCTIONS, VALID_AUTOMATION, VALID_AUTHORITY, VALID_RISK, VALID_STAGE)
3. Update every skill in `skills/` that references the changed enum
4. Update every matching codex skill in `codex/skills/`
5. Update every rule in `rules/` that documents the enum
6. Update any meta-EOU in `engine/meta-eous/` that uses the value

Never update vocabulary in only one place — all six layers must stay in sync.

### Testing changes

After editing any script, run the validator against the fixture foundry:

```bash
python3 scripts/validate_foundry.py tests/fixture-foundry
```

After editing any SKILL.md, verify the output path it declares matches the path the consuming skill (`eou-promote`, `eou-diagnose`, `eou-refactor`, `foundry-audit`, `audit-candidate-eou-set`) reads from.

### Skill selection guide

| Goal | Use |
|------|-----|
| Synthesize Stage 0 inputs from references (literacy bridge + per-app constitutional layer) | `$generate-captured-workflow-from-references` |
| Convert workflow → candidate EOUs | `$generate-eou-candidates` (consults `captured_workflow` when present per Rule 96) |
| Audit a candidate set for readiness | `$audit-candidate-eou-set` (runs Set Value Coverage Test when `captured_workflow` present) |
| Convert approved candidate → spec | `$eou-specify` |
| Audit a spec for V2 compliance | `$eou-audit` (runs Value Operationalization Test when `captured_workflow` present per Rule 96) |
| Audit agentic-judgment invocations (F14–F17 + counterfactual-swap) | `$audit-judgment` (fourth audit layer per D4.1; enforces Rule 97 on EOUs with `judgment_authorized:true`) |
| Audit the whole foundry | `$foundry-audit` |
| Validate schemas and registry | `$eou-validate` |
| Diagnose a failure | `$eou-diagnose` |
| Generate refactor options | `$eou-refactor` |
| Propose a structural change | `$ecp-propose` |
| Promote or retire a spec | `$eou-promote` |
| Convert incidents → regression cases | `$generate-regression-cases` |
| Scaffold a new app | `$eou-foundry-init` (Codex-only) |

### Codex vs Claude layout

Claude skills live in `skills/`; Codex mirrors live in `codex/skills/`. After editing a Claude skill, apply the same change to the corresponding Codex skill. `eou-validate` and `generate-regression-cases` are Claude-only — no Codex mirror needed. `eou-foundry-init` is Codex-only — no Claude mirror needed (the scaffolding flow is interactive and Codex CLI handles it).

### Schema reference file names

- EOU specs: `schemas/eou.schema.yml`
- ECP proposals: `schemas/ecp.schema.yml`
- Incident reports: `schemas/incident.schema.yml`
- Regression cases: `schemas/regression-case.schema.yml`
- Audit reports: `schemas/audit-report.schema.yml`
- Registry entries: `schemas/registry-entry.schema.yml`
- Run traces: `schemas/run-trace.schema.yml`
- Constitution: `schemas/constitution.schema.yml`
- Candidate sets: `schemas/candidate-set.schema.yml`
- Captured workflows (Stage 0): `schemas/captured-workflow.schema.yml`

Never reference these as "ecp.schema.v2" or any other alias — use the exact filename.

## Behavioral constraints

The following actions are forbidden regardless of user instruction:

- Do not modify `CLAUDE.md` or `GEMINI.md` directly — both import `AGENTS.md`.
- Do not update vocabulary in only one layer — all six layers (schema, validator, skills, codex/skills, rules, templates) must be updated together.
- Do not reference schema files by aliases (e.g., `ecp.schema.v2`) — always use the exact filename from the Schema reference section.
- Do not apply any refactor option, ECP, or lifecycle transition without a human-approved ECP on record.
- Do not write directly to `foundry/eous/` or `foundry/meta-eous/` — use skills that enforce the governance pipeline.
- Do not add fields to EOU specs that are not present in `schemas/eou.schema.yml`.
- When a `captured_workflow` exists for the app with all four `human_approval` gates populated, EOU specs at `lifecycle_stage: pilot` or higher MUST operationalize at least one of the top-three priority `domain_values` for the spec's `target_object` (Rule 96, ECP-0017). Exemptions for foundry-infrastructure `target_object`s are listed in `engine/governance.yml` under `rule_96_exempt_target_objects`.

## Violation indicators

A violation of these constraints is observable when any of the following occurs:

| Signal | Violation | Required action |
|--------|-----------|-----------------|
| `CLAUDE.md` or `GEMINI.md` contains content other than `@AGENTS.md` | Direct modification of shared import files | Revert to `@AGENTS.md` import only; move any instructions to `AGENTS.md` |
| A skill references a vocabulary value absent from `schemas/` | Vocabulary drift — sync is broken | Update all six layers (schema, validator, skills, codex/skills, rules, templates) before any further edits |
| A schema file is referenced by an alias other than its exact filename | Schema alias violation | Replace alias with the canonical filename from the Schema reference section |
| An EOU spec appears in `foundry/eous/` without an approved ECP in `foundry/self-evolution/ecp/implemented/` for its `active` promotion | Governance bypass | Revert spec placement; open an ECP and complete the full governance pipeline |
| `scripts/validate_foundry.py` reports `VALID_FUNCTIONS` containing a value not in `schemas/eou.schema.yml` | Validator out of sync | Reconcile `scripts/validate_foundry.py` constants against `schemas/eou.schema.yml` before any vocab-dependent work |

## Shared Memory

**Always write new instructions, rules, and memory to `AGENTS.md` only.**

Never modify `CLAUDE.md` or `GEMINI.md` directly — they only import `AGENTS.md`.
This keeps Claude Code, Codex CLI, and Gemini CLI on the same context.

## Project Structure

- `.claude/` — Claude Code skills, agents, rules, hooks, commands
- `.agents/skills/` — symlink to `.claude/skills/` (Codex skill scan path)
- `.codex/prompts/` — Codex slash-command prompts
- `.codex/hooks.json` / `.codex/config.toml` — Codex hooks/config (optional)
- `.gemini/skills/`, `.gemini/commands/` — Gemini skills and TOML commands
- `.mcp.json` — MCP server registrations (shared by all three tools)

---
> Source: [xiaolai/eou-foundry](https://github.com/xiaolai/eou-foundry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
