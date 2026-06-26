## document-generation-system

> This file is the project-level operating guide for AI agents working in this repository.

# Agent Operating Guide

This file is the project-level operating guide for AI agents working in this repository.

## Core Language Policy

- Chat with the user must be in Simplified Chinese.
- Repository files must be written in English.
- Code comments, documentation, task cards, test names, commit message drafts, and generated artifacts must be English.
- Existing Chinese text discovered during implementation should be treated as a remediation item.
- Localized UI resource files may contain localized strings only when they are intentionally part of product i18n.

## Planning and Execution Roles

GPT-5.5 owns:

- Review quality.
- Planning and orchestration.
- Task decomposition.
- Acceptance criteria.
- Risk classification.
- Final review after implementation batches are complete.

Lower-tier implementation models own:

- Narrow task execution.
- Small test additions.
- Mechanical refactors with clear instructions.
- Documentation updates from an explicit outline.

Lower-tier models must not:

- Make broad architecture decisions.
- Change authentication or migration strategy without a task card.
- Add dependencies without approval.
- Expand scope across workstreams.
- Guess product behavior when requirements are ambiguous.

## Required Starting Point

Before implementation, read these files:

1. `docs/audits/full-project-review-2026-04-26/README.md`
2. `docs/audits/full-project-review-2026-04-26/08-readiness-review.md`
3. `docs/audits/full-project-review-2026-04-26/09-task-cards.md`
4. `docs/audits/full-project-review-2026-04-26/10-implementation-review-checklist.md`
5. `docs/audits/full-project-review-2026-04-26/11-execution-sequence.md`
6. `docs/audits/full-project-review-2026-04-26/12-skill-registry.md`

## Project Skills

Use project skills from `.cursor/skills/` when the task matches the skill description:

- `project-remediation-runner`: required for all remediation task cards.
- `backend-security-hardening`: backend security, SSRF, callbacks, Actuator, and secrets.
- `docxtemplater-service-hardening`: Node rendering service, sandbox, routes, and Java-to-Node contracts.
- `frontend-vue-workspace`: Vue workspace, OnlyOfficeEditor, VersionDiffPanel, SegmentVersionDialog, and frontend tests.
- `delivery-governance`: CI/CD, Docker, release runbooks, and dirty working tree classification.
- `migration-safety`: Flyway, PostgreSQL upgrade planning, and `segment_versions` migration risk.
- `git-change-management`: git status review, commit grouping, and review-ready handoff.
- `local-deployment-operations`: local Docker Compose deployment and smoke testing.
- `validation-test-runner`: scoped validation commands and result reporting.

## Implementation Rules

- Work on exactly one task card at a time.
- Respect files in scope and files out of scope.
- Add or update tests required by the task card.
- Run the validation commands listed in the task card when possible.
- Update `07-iteration-log.md` after meaningful implementation or validation.
- Update `05-traceability-matrix.md` when a tracked item changes state.
- Stop instead of guessing if a task card stop condition is reached.

## Git and File Safety

- Do not revert user changes.
- Do not run destructive Git commands.
- **Commit cadence (repository owner preference):** After completing each remediation **task card** (or a single coherent batch for one card), create **one** `git commit` unless the user opts out for that session. Use an **English** message and include the **task ID** in the subject when applicable (for example `WS-02-T05: Add POST /watermark route`). Do **not** `git push` unless the user explicitly requests it. Skip committing when there are no meaningful changes, or when files are secrets, local-only credentials, or generated outputs (for example `backend/target/`, `node_modules/`).
- For ad-hoc requests to commit the **entire** working tree, still exclude secrets and build artifacts; prefer reviewing `git status` before staging.
- Do not delete untracked files unless a task explicitly asks for cleanup.
- Treat the current working tree as user-owned.
- Use `git-change-management` before preparing commits or grouping completed work.

## Local Deployment Safety

- Use `local-deployment-operations` for local Docker Compose work.
- Do not use production secrets locally.
- Do not connect local validation to production services.
- Do not delete Docker volumes unless explicitly requested.

## Validation Safety

- Use `validation-test-runner` after implementation tasks.
- Run only validation commands relevant to the active task card.
- Record commands not run and the reason.

## Security Rules

- Do not introduce unauthenticated write paths.
- Do not trust external URLs without validation.
- Do not log secrets or full token-bearing URLs.
- Do not add default production secrets.
- Do not weaken tenant isolation.
- Do not modify Flyway migrations unless the task is explicitly a migration task.

## Completion Response Format

When finishing a task, report:

- Task ID.
- Files changed.
- Tests added or updated.
- Validation commands run and results.
- Any commands not run and why.
- Remaining risks.
- Whether any stop condition was encountered.

---
> Source: [bigballgis/Document-Generation-System](https://github.com/bigballgis/Document-Generation-System) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
