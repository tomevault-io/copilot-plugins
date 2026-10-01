## sondar

> - Keep runtime state, fetched source data, JSONL signals, and generated reports under `.sondar/`.

# Sondar Repository Instructions

## Local Data

- Keep runtime state, fetched source data, JSONL signals, and generated reports under `.sondar/`.
- Never commit `.sondar/`, `.env*`, `.gstack/`, `sondar-workspace/`, `data/`, `reports/`, `bin/`, or `dist/`.
- Keep public examples free of credentials, personal filesystem paths, and generated user data.

## Engineering Constraints

- Keep production code and tests in Go. Do not add Python scripts, Python tests, Python caches, or Python-based build and validation steps.
- Use shell only for thin operational wrappers such as release packaging.

## Release Policy

- Use semantic versions. Git tags must be annotated and named `vX.Y.Z`.
- Keep the version identical across `Makefile`, the matching `CHANGELOG.md` heading, the built binary, and the Git tag. `Makefile` stores `X.Y.Z` without the `v` prefix.
- For every release, add a dated `## [X.Y.Z]` entry to `CHANGELOG.md` before tagging.
- Run `make release-check` and require a clean working tree before creating a release tag.
- Commit the version and changelog first, then create the tag on that release commit:

  ```bash
  git tag -a vX.Y.Z -m "Sondar vX.Y.Z"
  git push origin main
  git push origin vX.Y.Z
  ```

- Let `.github/workflows/release.yml` build and publish release archives and `SHA256SUMS`; do not upload ad hoc binaries.
- Never move, reuse, or force-push a published version tag. Publish a new patch version for corrections.

---
> Source: [geekjourneyx/sondar](https://github.com/geekjourneyx/sondar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
