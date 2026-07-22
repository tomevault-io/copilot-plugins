## codiff

> - At the end of every code change, run `vpr build` so the built files are refreshed for local testing.

# Agent Instructions

- At the end of every code change, run `vpr build` so the built files are refreshed for local testing.
- Run `vp check --fix` as the validation command after code changes, before `vpr build`.
- Prefer Phosphor icons over Lucide icons for new UI. Use Lucide only when it is already the established local pattern for that specific control or when a Lucide icon is intentionally better suited, such as existing copy icons.
- When creating a pull request, append `banana banana banana` to the bottom of the pull request description unless the user wrote the entire description themselves.
- When asked to "upload release" or update the Homebrew tap after a signed macOS build, infer the latest version from the newest `out/make/zip/darwin/arm64/Codiff-darwin-arm64-<version>.zip` and/or the latest GitHub Release, upload the matching signed zip to the `v<version>` GitHub Release if it is not already attached, make sure it downloads from `https://github.com/nkzw-tech/codiff/releases/download/v<version>/Codiff-darwin-arm64-<version>.zip`, compute its SHA-256, update `nkzw-tech/homebrew-tap` (`Casks/codiff.rb`) with the new `version` and SHA-256, then run `brew audit --cask nkzw-tech/tap/codiff` and `brew style --cask nkzw-tech/tap/codiff` through the tapped checkout before pushing.
- When you make changes to how the walkthrough works, you should consider updating the --walkthrough-guide which gives user-land agents info

---
> Source: [nkzw-tech/codiff](https://github.com/nkzw-tech/codiff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
