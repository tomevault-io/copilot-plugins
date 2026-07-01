## local-workflow-skill

> This repository is the source workspace for Portable Agent Workflow: a generic,

# AGENTS.md

This repository is the source workspace for Portable Agent Workflow: a generic,
Markdown-first workflow framework and curated skill library for multiple coding
agents.

## Repository Workflow

For substantial work, follow the Four-State Workflow:

```text
scan -> state restore -> intent -> authority -> route -> execute -> validate -> sync -> deliver -> evolve
```

Read these core files before changing workflow, adapter, skill, or state
behavior:

- `framework/core/01-workflow.md`
- `framework/core/02-state-systems.md`
- `framework/core/03-routing.md`
- `framework/core/04-authority.md`
- `framework/core/05-validation.md`
- `framework/core/06-evolution.md`
- `framework/core/07-artifact-contracts.md`
- `framework/core/08-autonomous-project-loop.md`

## State System

This repository uses local Four-State files:

- `state/LOG.md`
- `state/REQUIREMENTS.md`
- `state/MEMORY.md`
- `state/PROGRESS.md`

Update them when repository truth changes. Prefer appending concise entries
instead of rewriting history.

Before substantial work, produce a `State Restore` summary from these files:
state sources, stale/placeholders, latest goal, latest validation, blocker, and
current assumptions. If placeholders or stale entries remain, replace what can
be confirmed from repository facts before editing workflow behavior.

## Artifact Contracts

Do not leave substantial audit, acceptance, implementation, fix, validation, or
docs-sync work only in chat.

- For `audit` / `acceptance` / `验收预检`, create or update a report under
  `docs/audit/` when the target project has that directory, and update
  `docs/audit/INDEX.md`.
- For implementation or fixes, update the relevant state files, especially
  `state/LOG.md` and `state/PROGRESS.md`; update docs when behavior, usage,
  architecture, validation, or risk changes.
- For docs changes, update `docs/INDEX.md` or the relevant directory index so
  new files are discoverable.
- Before commit, confirm validation evidence, state sync, index sync, and
  `git status`; split multiple logical changes into multiple atomic commits.

## Autonomous Project Progress

When the user asks to continue a project, move forward by plan, auto-audit and
fix, or otherwise make real project progress, first check whether the project
has a usable roadmap and next-action plan. If not, create or update those
planning artifacts before implementation. Then follow
`framework/core/08-autonomous-project-loop.md`: select one small goal, confirm
acceptance criteria, implement, validate, self-audit, repair in-scope findings,
  sync state/docs, commit when eligible, and either select the next goal or stop
with a clear reason.

Every autonomous loop, audit/fix loop, important implementation, or handoff
must leave a `Loop Record` in `state/PROGRESS.md` or `state/LOG.md`: goal,
acceptance criteria, validation evidence, self-audit, repairs, state/docs sync,
commit, next goal, and stop reason.

## Skill Library Policy

This is a generic open-source skill/workflow repository.

- Keep generic skills and semi-generic vendor/product skills.
- Do not add project-specific skills tied to private projects, product names,
  local paths, accounts, servers, or one-off command wrappers.
- Store project-specific skills in the target project's local `.codex/skills`,
  `.claude/skills`, `.agents/skills`, or a private repository instead.
- Preserve license and provenance files for imported third-party material.

## Delivery Discipline

- Inspect status and diffs before staging or committing.
- Do not stage unrelated local changes.
- Run validation appropriate to the scope.
- In a Git repository, every completed and validated change must enter the
  commit closure unless the user explicitly says not to commit.
- Each commit must represent one complete work package, including required
  tests, docs, and state updates.
- Split multiple logical changes into multiple atomic commits before delivery.
- Stage by explicit paths; do not use `git add .` or `git add -A` as a shortcut.
- If commit is blocked by missing Git, failed validation, conflicts, sensitive
  files, or a user decision, record the reason and next step.
- Do not auto-push, merge, or open a PR unless explicitly requested.

---
> Source: [cass-2003/local-workflow-skill](https://github.com/cass-2003/local-workflow-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
