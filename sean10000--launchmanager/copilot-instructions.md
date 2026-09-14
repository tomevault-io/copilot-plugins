## launchmanager

> When publishing LaunchManager:

# Release workflow

When publishing LaunchManager:

- **Do not** manually edit `MARKETING_VERSION` or `CURRENT_PROJECT_VERSION` in `project.pbxproj`.
- **Do not** create `scripts/RELEASE_NOTES_v*.md` files.
- **Do** add changes under `## [Unreleased]` in `CHANGELOG.md` during development.
- **Do** run `./scripts/release.sh patch` (or `minor` / `major` / explicit version) to bump `Version.xcconfig`, finalize the changelog, commit, tag, and push.

CI (`.github/workflows/release.yml`) handles building the DMG, GitHub Release, and Homebrew tap update.

For local test builds only: `./scripts/build-dmg.sh`.

---
> Source: [Sean10000/LaunchManager](https://github.com/Sean10000/LaunchManager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
