## betterbot

> BetterBot currently consists of `BetterBot Deployment Plan.pdf`, which stays the canonical roadmap. When implementation starts, create `src/` for runtime code, `tests/` mirroring its package layout, and `docs/` for living specs extending the deployment plan. Group assets (diagrams, datasets, prompts) under `assets/` and keep the repository root free of loose files.

# Repository Guidelines

## Project Structure & Module Organization
BetterBot currently consists of `BetterBot Deployment Plan.pdf`, which stays the canonical roadmap. When implementation starts, create `src/` for runtime code, `tests/` mirroring its package layout, and `docs/` for living specs extending the deployment plan. Group assets (diagrams, datasets, prompts) under `assets/` and keep the repository root free of loose files.

## Build, Test, and Development Commands
No automated toolchain ships yet. Once you add code, provide a `Makefile` (or language runner) exposing `make setup`, `make lint`, and `make test` so every contributor can bootstrap and validate in one step. Mirror the same commands with language-native scripts—`poetry install`, `npm run lint`, etc.—and document prerequisites near the commands.

## Coding Style & Naming Conventions
Default to 4-space indentation, UTF-8 text, and a 100-character line limit. Name modules and packages with `snake_case`, classes with `PascalCase`, and public functions with action verbs. Adopt the formatter and linter that fit the chosen language (Black+Ruff for Python, Prettier+ESLint for TypeScript) and surface them behind a `make format` target.

## Testing Guidelines
Place fast unit tests in `tests/`, mirroring the production module path (e.g., `tests/services/test_scheduler.py`). Keep fixtures under `tests/fixtures` and favour isolated, deterministic cases. Aim for ≥80% branch coverage before opening a pull request and run `make test` locally plus in CI to guard against regressions.

## Commit & Pull Request Guidelines
History currently uses short imperative summaries (`start`), so continue writing concise, lowercase subjects ≤50 characters (e.g., `add task scheduler`). Expand in the body with motivation, risk areas, and follow-up TODOs. Each pull request should link the relevant plan item, outline functional changes, list validation steps (commands or screenshots), and note any new configuration or secrets reviewers must provision.

## Documentation Workflow
Update `BetterBot Deployment Plan.pdf` whenever scope or dates shift, and export a Markdown digest to `docs/` so reviewers can diff textual edits. Store diagrams or UI flows under `assets/` in SVG or PNG form and reference them from Markdown. If document generation becomes scripted, add a `make docs` target and record dependencies such as `pandoc`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Galjoentopper)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Galjoentopper)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
