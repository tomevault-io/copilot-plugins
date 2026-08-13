## github

> This repository defines shared Codex governance defaults for MDSoftware-DE.

# AGENTS.md

This repository defines shared Codex governance defaults for MDSoftware-DE.

## Scope
- Shared issue templates and contribution defaults
- Shared AGENTS baseline and template
- Shared skill package for Codex governance alignment

<!-- MD-ORG-AGENTS-BASELINE:START -->
## MDSoftware-DE Org Baseline

Default language: English.

Required rules:
- Use English for documentation, issues, pull requests, and operational run notes.
- Do not store secrets, passwords, tokens, or private keys in repository files.
- Keep project-specific operations and architecture guidance in this file.
- For workflow/process changes, document: Purpose, Current Status Snapshot, Last Change, Quick Test, Maintenance Rule.
- Keep docs structure consistent: include `docs/README.md` (or `docs/index.md`), `docs/diagrams/`, `docs/runbooks/`, `docs/architecture/`, and `docs/AGENTS.md`.
- Run every feasible Linux security, policy, quality, test, and build job on Wolverine with labels containing both `self-hosted` and `wolverine`.
- Never add an automatic fallback to GitHub-hosted runners and never disable a required security or quality gate to avoid billing.
- Central reusable workflows in `MDSoftware-DE/.github` must default to `["self-hosted","wolverine"]`; repository-specific labels may only refine that Wolverine pool.
- Treat a queued job as a Wolverine capacity or availability incident. Do not silently change it to `ubuntu-latest`.
- Permit Windows, macOS, or other hosted execution only through an exact centrally documented exception with owner, approval reference, and review date.
- Remember that `actions/upload-artifact` remains GitHub-managed storage and Dependabot remains a GitHub service; migrate those through the separately approved artifact-store and Renovate phases.
- Mermaid authoring baseline:
  - Never use `\\n` in Mermaid labels; use normal spaces.
  - Flowcharts in `docs/diagrams/*` should use `classDef` + `class`/`style` color mapping.
  - State diagrams in `docs/diagrams/*` must define at least 3 semantic color groups with `classDef`, map states using grouped `class`/`style` assignments, use semantic class names (for example: `entry`, `active`, `review`, `success`, `error`, `terminal`), and should prefer explicit state aliases plus inline `:::class` markers for GitHub compatibility.
  - Sequence diagrams must use `autonumber` or explicit contiguous numeric prefixes (`1.`, `2.`, ...), plus colored visual grouping (`rect` or `box`).
  - ER diagrams should use semantic color mapping (`classDef default`, semantic `classDef` + grouped `class`/`style`, or explicit `style` mapping) and keep ER style tokens GitHub-safe (`fill`, `stroke`, `color`; avoid `stroke-width`/`font-weight`).
- Keep this block unchanged so org automation can verify baseline adoption.

<!-- MD-ORG-AGENTS-BASELINE:END -->

## Operational Notes
- Canonical baseline block: `docs/codex/AGENTS_BASELINE_BLOCK.md`
- Canonical template for new repos: `docs/codex/AGENTS_TEMPLATE.md`
- Canonical standards doc: `docs/codex/CODEX_ORG_STANDARDS.md`
- Canonical skill: `codex/skills/org-agents-governance/SKILL.md`

## Lessons
- When GitHub-hosted billing is disallowed, keep required checks enabled and route feasible workflows to Wolverine through self-hosted runner labels instead of disabling security gates or adding a hosted fallback.
- When a hand-written unified diff needs multiple logical edits, validate and apply one exactly counted hunk at a time so one malformed hunk cannot block or obscure the others.
- Do not use Markdown hard-break trailing spaces in governed documents; use separate paragraphs or explicit structure so `git diff --check` remains clean.
- Before relying on `cp --update=none` for a no-clobber backup, verify that the installed `cp` supports the argument; otherwise require an explicit absent-target check before plain `cp`.
- When introducing an enforcement validator before its migration change, record the exact pre-migration failures as the expected red state and require the migration task to turn that same check green.
- Verify the actual minimum Python interpreter before writing plan signatures; use `Optional[Sequence[str]]` instead of PEP 604 unions when Python 3.8 must parse the file.

---
> Source: [MDSoftware-DE/.github](https://github.com/MDSoftware-DE/.github) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
