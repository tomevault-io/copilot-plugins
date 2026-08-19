## wt

> - `CHANGELOG.md` covers user-visible changes to `wt`. Omit CI, release-process, and other repository-maintenance changes unless they alter the installed tool's behavior.

# Project notes

## Changelog

- `CHANGELOG.md` covers user-visible changes to `wt`. Omit CI, release-process, and other repository-maintenance changes unless they alter the installed tool's behavior.
- Keep entries brief and update `Unreleased` as user-facing tasks finish.
- Before releasing, review `Unreleased`, consolidate related entries, move them under the new version, and start a fresh `Unreleased` section.

## Release

- From a clean, synchronized `main`, run `mise run release`.
- It tests, advances the `0.x` version, tags, and pushes. GitHub builds macOS, Linux, and Windows archives, publishes the release, and updates `mikker/homebrew-tap`.

---
> Source: [mikker/wt](https://github.com/mikker/wt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
