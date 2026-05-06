## agent-sessions

> 1. Bump version in `src-tauri/tauri.conf.json` and `package.json`

## Releasing a new version

1. Bump version in `src-tauri/tauri.conf.json` and `package.json`
2. Update `CHANGELOG.md` with the new version and changes
3. Commit and push
4. Run `source .env && scripts/release.sh`

The `.env` file contains Apple credentials (`APPLE_ID`, `APPLE_PASSWORD`, `APPLE_TEAM_ID`) needed for notarization.

The release script builds both aarch64 and x64, signs, creates DMGs, notarizes with Apple, publishes a GitHub release, and updates the Homebrew tap (`ozankasikci/homebrew-tap`).

If a build already exists, use `--skip-build` to only sign/notarize/publish. Other flags: `--skip-notarize`, `--skip-github`, `--skip-homebrew`, `--arch aarch64|x64`.

---
> Source: [ozankasikci/agent-sessions](https://github.com/ozankasikci/agent-sessions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
