## px

> - Keep commit messages to a single line (no multi-line messages).

# Agent Guidelines for Px

## Git commits

- Keep commit messages to a single line (no multi-line messages).
- Do not add "Generated with" lines, "Co-Authored-By" lines, or any AI tool attribution to commit messages.

## Before pushing to GitHub

- Test all affected configurations locally (if possible) before pushing to GitHub.
- Cancel all old/running jobs on GitHub Actions before pushing new changes.
- Monitor jobs after pushing until they complete and confirm they pass.
- Do not wait for all jobs to finish before starting fixes — use `gh run view --log-failed` to fetch failure logs as soon as a job fails, diagnose immediately, and push fixes as soon as you have a clear picture of all failing cases.

## Documentation

Px has both **user-facing** and **developer-facing** documentation. Both must be kept current.

### User documentation

- Update `docs/installation.md` when install methods, dependencies, Docker images, or binary packaging change.
- Update `docs/usage.md` when CLI flags, configuration options, credential handling, examples, or limitations change.
- Update `docs/configuration.md` when config sources, option defaults, or auth types change.
- Keep `README.md` in sync — it contains a brief overview with links to the docs folder. Links must use full `https://github.com/genotrance/px/blob/master/docs/` URLs since the README is also displayed on PyPI and Docker Hub.

### Developer documentation

- Update `docs/architecture.md` when the code structure, runtime model, module responsibilities, or data flow change.
- Update `docs/build.md` when the build system, `pyproject.toml`, GitHub Actions workflows, or release process change.
- Update `docs/testing.md` when the test suite structure, fixtures, or coverage configuration change.
- Update `docs/changelog.md` with every user-visible change.

### Changelog

- Each release in `docs/changelog.md` should separate **user-facing** changes (under `Bug fixes`, `New features`, `Improvements`) from **internal** changes (under `Internal`). Users should not need to read about tooling, CI, or refactoring unless it affects them directly.

### General principles

- Docs should explain **what was done and why** so they serve as a future reference of how the project evolved and how to use the project effectively.
- Do not copy file contents (config snippets, YAML, TOML) verbatim into docs — refer to files by name and describe the intent instead.
- When adding a new feature or changing behaviour, update both the user docs (so users know how to use it) and the developer docs (so contributors understand the implementation).

## Scope discipline

- Do not remove any capability or support unless explicitly asked by the user.
- Do not weaken or delete tests without explicit direction.

## Test coverage

- Make sure test cases test all features and configurations of the project.
- When adding new features or fixing bugs, add or update tests to cover the new behaviour.
- All tests must pass on CPython 3.10–3.14 before merging.

---
> Source: [genotrance/px](https://github.com/genotrance/px) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
