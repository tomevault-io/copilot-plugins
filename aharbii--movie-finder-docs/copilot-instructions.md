## movie-finder-docs

> Foundational mandate for `movie-finder-docs` (`docs/`).

# Gemini CLI — docs submodule

Foundational mandate for `movie-finder-docs` (`docs/`).

---

## What this submodule does

MkDocs site with architecture diagrams and ADRs.

---

## Architecture diagrams

- **PlantUML:** Edit `.puml` in `architecture/plantuml/`.
- **Structurizr C4:** Edit `architecture/workspace.dsl`.
- **Constraint:** NEVER generate `.mdj` StarUML files programmatically.
- **Build:** Run `make mkdocs` from repo root — handles prepare-docs, PlantUML, and serve at :8001.

---

## ADR (Architecture Decision Records)

- Create in `architecture/decisions/` using the template in `index.md`.
- Status: `Proposed` → `Accepted`.

---

## Workflow invariants

- This repo is the gitlink path `docs` inside `aharbii/movie-finder`. Parent workflow/path
  filters must use `docs`, not `docs/**`.
- Cross-repo tracker issues originate in `aharbii/movie-finder`. Create the linked child issue in
  this repo only if this repo will actually change.
- Inspect `.github/ISSUE_TEMPLATE/*.yml`, `.github/PULL_REQUEST_TEMPLATE.md` when present, and a
  recent example before creating or editing issues/PRs. Do not improvise titles or bodies.
- For child issues in this repo, use `.github/ISSUE_TEMPLATE/linked_task.yml` and keep the
  description, file references, and acceptance criteria repo-specific.
- If CI, required checks, or merge policy changes affect this repo, update contributor-facing docs
  here and in `aharbii/movie-finder` where relevant.
- If a new standalone issue appears mid-session, branch from `main` unless stacking is explicitly
  requested.
- PR descriptions must disclose the AI authoring tool + model. Any AI-assisted review comment or
  approval must also disclose the review tool + model.

---

## VSCode setup

`docs/.vscode/` — workspace configuration for documentation editing.

- `settings.json`: PlantUML local renderer, Markdown word-wrap, ruler 120
- `extensions.json`: `jebbs.plantuml`, `yzhang.markdown-all-in-one`, `davidanson.vscode-markdownlint`
- PlantUML preview: `Option+D` / `Alt+D` with jebbs.plantuml extension
- Modifying configs: Update `CLAUDE.md`, `GEMINI.md`, `AGENTS.md`, and the repo's
  `.github/copilot-instructions.md` after any change.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aharbii)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/aharbii)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
