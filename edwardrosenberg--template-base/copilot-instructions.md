## template-base

> This is the **Tier 0 base template** for the organization. Every repository inherits from this template. Changes here propagate to all downstream stack (Tier 1) and archetype (Tier 2) templates via the platform sync workflow.

# Claude Code Instructions — template-base

This is the **Tier 0 base template** for the organization. Every repository inherits from this template. Changes here propagate to all downstream stack (Tier 1) and archetype (Tier 2) templates via the platform sync workflow.

## Repository Purpose

This repo contains **organization-wide governance only**: reusable CI/CD workflows, PR/issue templates, editor config, copilot/agent instructions, and CODEOWNERS. It has no application code, no language-specific tooling, and no build artifacts.

## What You Can Change

- `.github/workflows/` — Reusable workflows (`ci.yml`, `pr-title-lint.yml`, `agent-validation.yml`)
- `.github/copilot-instructions.md` — Agent guardrails (synced to all children)
- `.github/PULL_REQUEST_TEMPLATE.md`, `.github/ISSUE_TEMPLATE/` — PR and issue templates
- `.editorconfig`, `.gitignore` — Universal formatting and ignore rules
- `CODEOWNERS` — Review requirements
- `docs/` — Platform documentation

## What You Must NOT Change

- Do not add language-specific files (no `pom.xml`, `pyproject.toml`, `package.json`)
- Do not add application code or framework dependencies
- Every file here must be stack-agnostic

## Build & Validation

There is no build step for this repository. Validate changes by:

1. Checking YAML syntax for any modified workflow files
2. Confirming workflow inputs are backward-compatible with existing callers
3. Running the PR title lint regex against sample titles if modifying `pr-title-lint.yml`

## Key Conventions

- **Workflow inputs**: Use kebab-case for input names (e.g., `backend-tech-stack`)
- **File placement**: Config files at repo root, GitHub-specific files in `.github/`
- **Backward compatibility**: Reusable workflow changes must not break existing callers — add new optional inputs with defaults rather than changing existing ones

## Sync Awareness

Files listed in `sync-config.yml` → `sync_paths` are propagated to Tier 1 children. If you add a new governance file that all repos should have, add it to `sync_paths` as well.

Children of this repo:
- `template-backend-java` (Tier 1)
- `template-backend-python` (Tier 1)

## Versioning

This repo has no version file. It uses git tags (`vX.Y.Z`) on `main` for versioning, referenced by downstream callers (e.g., `@v1.0.0`).

When making changes, determine the appropriate version bump based on your change type and include a note in the PR description about which semver increment applies:

- **PATCH** — bug fixes, doc updates, non-breaking tweaks (`fix:`, `docs:`, `chore:`, `ci:`, `refactor:`, `test:`)
- **MINOR** — new features, new workflow inputs with defaults (`feat:`)
- **MAJOR** — breaking changes to workflow inputs or behavior (`feat!:`, `fix!:`, `chore!:`)

Breaking changes in this repo affect **every downstream template and project**. If your change breaks existing callers, document the migration path in the PR description and consider a deprecation period where old inputs still work alongside new ones.

## Branch Management

Every change must go through a pull request — never push directly to `main`.

1. **Create a branch** from `main` for each feature or fix:
   ```bash
   git checkout main && git pull
   git checkout -b <type>/<short-description>   # e.g. feat/add-workflow-input
   ```
2. **Commit and push** your changes to the new branch:
   ```bash
   git add -A && git commit -m "<type>(<scope>): <description>"
   git push -u origin HEAD
   ```
3. **Open a pull request** against `main` and wait for CI/CD to pass.
4. **Merge the PR** once CI is green and any required reviews are approved.
5. **Pull main** locally before starting the next change:
   ```bash
   git checkout main && git pull
   ```

Repeat for each subsequent feature or fix. Keep branches short-lived and focused on a single concern.

## PR Conventions

- Titles must follow Conventional Commits: `<type>(<scope>): <description>`
- Valid types: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `perf`, `ci`, `build`
- Keep PRs small and focused on a single concern
- No drive-by refactors or scope creep

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EdwardRosenberg)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/EdwardRosenberg)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
