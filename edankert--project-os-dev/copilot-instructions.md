## project-os-dev

> This repository is project-os enabled. Follow this startup sequence before doing any work.

# Agent Startup Contract

This repository is project-os enabled. Follow this startup sequence before doing any work.

## Mandatory First Steps
1. Read `CONTEXT.md`.
2. Read `docs/INDEX.md`.
3. Read `SNAPSHOT.yaml`.
4. Run `bash tools/agents/bootstrap.sh`.
5. Follow `tools/instructions/MARKDOWN.md`: do not hard-wrap Markdown prose to a fixed column width.

Do not skip or reorder these steps.

## Mandatory Docs-First Gate
Before any code edit, start documentation intake first:
1. Run `bash tools/agents/start-change.sh "<short title>"`.
2. Update the generated `docs/changes/CHG-*.md` note so every documentation type is explicitly marked.
3. Update `SNAPSHOT.yaml` for active work state.
4. Only then implement code changes.

After edits, run `bash tools/agents/check-docs-first.sh` and `bash tools/scripts/validate-docs.sh`, and fix any failures — the validator also runs at git pre-commit (`bash tools/scripts/install-git-hooks.sh` to install) and in CI, so drift becomes a build failure.

All documentation types must be considered on every behavior/path change:
- features
- requirements
- tasks
- issues
- tests
- workflows
- decisions (ADRs)
- risks
- changes
- snapshot

## Canonical State
- `SNAPSHOT.yaml` is the canonical current-work state.
- If note files and `SNAPSHOT.yaml` disagree, trust `SNAPSHOT.yaml` and report the mismatch.

## Primary Work Entrypoints
- Replace with project-specific commands after initialization.
- Optional docs cockpit: `bash tools/cockpit/run.sh docs --bind 127.0.0.1 --port 8765`.

## Edit Boundaries
- Live docs (frequent updates): `docs/features/**`, `docs/issues/**`, `docs/workflows/**`, `docs/changes/**`, `SNAPSHOT.yaml`.
- Reference docs (change rarely): `docs/ARCHITECTURE.md`, `docs/DESIGN.md`, `docs/STYLEGUIDE.md`, `docs/requirements/**`, `docs/risks/**`, `docs/decisions/**`, `docs/reference/**`, `docs/research/**`.
- Agent/tool operating instructions are not project documentation: `tools/**`.

## Output Expectations
- Provide a short project summary before edits: purpose, active feature/task/issue, intended files.
- After edits, report changed files and validation commands run.
- If blocked by missing dependencies or external repos, state exact missing path/tool.

---
> Source: [edankert/project-os-dev](https://github.com/edankert/project-os-dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
