## cetus

> - Before publishing a new release, increment the application version in both

# Repository instructions

## Releases

- Before publishing a new release, increment the application version in both
  `package.json` and `src-tauri/tauri.conf.json` and keep the values identical.
- A macOS or combined-platform release must use a version newer than the version
  currently published in `release/latest.json`.
- A Windows-only run may reuse the published version when it is adding or
  replacing the Windows asset in that same release.
- After publishing, verify that `release/latest.json`, the release asset names,
  and the `release` Git tag all identify the intended version and commit.

---
> Source: [drewnekota/cetus](https://github.com/drewnekota/cetus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
