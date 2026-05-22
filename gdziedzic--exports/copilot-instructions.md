## exports

> This repository is a collection of installable developer tools, configuration files, and Windows-focused utilities. Keep changes scoped, preserve existing layouts, and avoid introducing unnecessary build systems or repo-wide conventions that do not already exist here.

# AGENTS.md

This repository is a collection of installable developer tools, configuration files, and Windows-focused utilities. Keep changes scoped, preserve existing layouts, and avoid introducing unnecessary build systems or repo-wide conventions that do not already exist here.

## Repository Shape

- `README.md`: primary documentation for the repository contents and install flows.
- `repo-installer.json`: installation manifest; treat this as a source of truth for installable components.
- `Taskfile.yml`: lightweight task automation.
- `experiments/`: application or tool projects such as `DevChef` and `dao`.
- `dotnotfiles/`: user configuration assets such as Git and PowerShell settings.
- `xg/`: development utilities, currently including Taskfile extensions.
- `_manual/`: manually managed scripts and shortcut/config exports.
- `prompts/`: authoring prompts and supporting text assets.

## Working Rules

- Match the style and granularity of the area you are editing. This repo is intentionally heterogeneous.
- Prefer minimal, surgical edits over broad cleanup.
- Do not rename top-level directories or reorganize installable assets unless explicitly requested.
- Preserve Windows compatibility for scripts, paths, and instructions unless a task clearly targets cross-platform behavior.
- When updating docs, keep examples aligned with the current repository structure and installer workflow.
- When changing `repo-installer.json`, verify referenced paths and component names still match the filesystem.

## Common Commands

Run commands from the repository root unless a component requires otherwise.

```powershell
python repo-installer.py list
python repo-installer.py status
python repo-installer.py install --dry-run
task start_server
```

Component-specific work may require entering a subdirectory under `experiments/` or `xg/`.

## Editing Guidance

- Documentation changes should usually include `README.md` updates when user-facing behavior changes.
- Config updates in `dotnotfiles/` should preserve user intent and existing platform assumptions.
- Script changes under `_manual/` or `xg/` should keep dependencies explicit and avoid hidden environment assumptions.
- Avoid adding large generated files, lockfiles, or vendored artifacts unless the task explicitly requires them.

## Validation

Use the lightest validation that fits the change:

- For docs-only changes, review formatting and path references.
- For Taskfile changes, ensure task names and commands remain runnable from the repo root.
- For installer or manifest changes, run a dry-run install flow when possible.
- For subprojects in `experiments/`, use the validation method local to that project instead of inventing a new top-level workflow.

## Notes For Agents

- Check for existing uncommitted changes before editing and avoid overwriting user work.
- If a task touches multiple areas, document cross-directory assumptions clearly.
- Prefer updating existing docs over creating redundant new documentation files.

---
> Source: [gdziedzic/exports](https://github.com/gdziedzic/exports) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
