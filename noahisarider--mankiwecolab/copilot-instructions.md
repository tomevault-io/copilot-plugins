## mankiwecolab

> Instructions for AI agents working on this repository.

# AGENTS.md

Instructions for AI agents working on this repository.

## Project

**Mankiwecolab** — interactive economics experiments based on Mankiw's *Principles of Economics*. Python package (`mankiwecolab` on PyPI), CLI entry point `mankiw-econ`. CI (`.github/workflows/ci.yml`) runs tests + ruff on every push.

## ⚠️ Release Process (read before touching versions)

**The publish workflow ONLY triggers on tag pushes.** A commit alone — even with a bumped version — does NOT create a GitHub Release or update PyPI. If you commit a version bump without a tag, the release is silently skipped.

Correct flow, in order:

1. **Bump the version** in `pyproject.toml` (`version = "x.y.z"`, semver).
2. **Commit** the change (e.g. `Bump version to x.y.z`).
3. **Create and push a tag** — this is the trigger:
   ```bash
   git tag vX.Y.Z
   git push origin vX.Y.Z
   ```
4. `.github/workflows/publish.yml` then automatically: builds the package → publishes to PyPI (trusted publishing, no token needed) → creates a GitHub Release with wheel + sdist + attestations.

## Verify a release actually happened

After pushing the tag, confirm all three:

1. Workflow run: `gh run list --workflow publish.yml` → `completed / success`
2. GitHub Release: `gh release view vX.Y.Z` → assets include `mankiwecolab-X.Y.Z-py3-none-any.whl` and `.tar.gz`
3. PyPI: `curl https://pypi.org/pypi/mankiwecolab/json` → `info.version == X.Y.Z`

If the workflow failed, check the run logs (most common causes: version already published on PyPI, or tag name ≠ `v*`).

## Rules

- **Git identity must be** `NoahIsARider <noahisarider@users.noreply.github.com>` — never commit as anyone else.
- Do not push a tag without bumping the version first (PyPI will reject a duplicate version).
- Do not bump the version without pushing the tag (release will silently not happen).
- When testing an install right after release: the Tencent Cloud PyPI mirror (`mirrors.tencentyun.com`) lags by hours — use `-i https://pypi.org/simple` for immediate verification.
- Package name is `mankiwecolab` (all lowercase) — not `mankiw-economics`, not `MankiwEcoLab`.

---
> Source: [NoahIsARider/MankiwEcoLab](https://github.com/NoahIsARider/MankiwEcoLab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
