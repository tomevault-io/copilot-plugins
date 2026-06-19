## kivun-terminal-wsl

> **When the bulletproof option is obvious, do it. Do not ask.**

# Working notes for AI agents on this repo

## Decision-making

**When the bulletproof option is obvious, do it. Do not ask.**

If you have validated half of a feature (e.g. the decline path of an installer prompt) and the other half is obviously the path users actually care about (e.g. the auto-install path itself), add the validation - don't offer the user a choice between the obviously-right and obviously-wrong option. Framing an obvious call as a question shifts engineering judgment onto the user and risks them picking the worse option for the wrong reason (saving CI minutes, looking decisive, etc.).

Reserve questions for genuine trade-offs - cost vs. benefit, scope, taste. Completeness and self-doubt are not the same as helpfulness when the answer is obvious.

## Bulletproofing this product specifically

This repo ships a launcher that runs on someone else's Windows machine. The user-visible failure mode that ate v1.1.0 was: launcher said "Claude not found", then claimed to fall back, then crashed running the missing binary. Treat every launcher path as a path that must work end-to-end on a clean machine - not just "exit cleanly when broken." The CI in `.github/workflows/validate-launcher-windows.yml` exists to enforce this; if you add a new launcher branch, add a CI job that exercises it against real WSL.

## Workflows

### Automatic Publishing

When publishing a release on this repo:

1. **Versioning** — bump `VERSION` at the repo root (the source of truth — there is no `package.json`). Add a corresponding entry in `docs/CHANGELOG.md`. Optionally add `docs/RELEASE_NOTES_<version>.md` for a user-friendly GitHub release page; CI prefers it over the CHANGELOG section.
2. **Binary Integrity** — never upload manually built artifacts. The release pipeline is tag-push driven: pushing `vX.Y.Z` triggers `.github/workflows/build-windows.yml` (Windows installer) and `.github/workflows/build-linux.yml` (Linux tarball), each rebuilds clean and attaches via `softprops/action-gh-release@v2`.
3. **Atomic Release** — push the version-bump commit to the branch FIRST, then push the tag: `git push origin vX.Y.Z`. The GitHub release is created by the tag push, not by the branch merge.
4. **Asset Persistence (badge re-sync)** — if a release exists but the Shields.io downloads badge shows "invalid" or "no data", run `gh release upload v<version> <asset> --clobber` against an existing asset to force GitHub to refresh the asset metadata; this tickles the API's `download_count` field which the badge cache reads. Use this whenever a user reports the badge stuck on "invalid". (Verbatim phrase to expect from the user: "Refresh the release assets for the latest version.")

---
> Source: [noambrand/kivun-terminal-wsl](https://github.com/noambrand/kivun-terminal-wsl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
