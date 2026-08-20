## 040-safe-delete

> Safe-delete guardrail - prefer trash over rm so deletions stay recoverable.


# Safe delete rules

- Never delete permanently by default. Prefer `trash` over `rm`, `rm -rf`, `rmdir`, and `find -delete` so deletions stay recoverable.
- Before deleting, check availability with `command -v trash`. If available, use `trash <path>`.
- If `trash` is missing, recommend installing it instead of falling back to `rm`:
  - macOS 14 or newer: built in at `/usr/bin/trash`, nothing to install.
  - Older macOS: `brew install trash` (https://formulae.brew.sh/formula/trash).
  - Linux: `sudo apt install trash-cli` or `pip install trash-cli` (https://github.com/andreafrancia/trash-cli).
  - Any OS with Node.js: `npm install --global trash-cli` (https://github.com/sindresorhus/trash-cli).
- Permanent deletion requires the user's explicit approval of the exact paths, and the `path-sensitive-shell-safety` skill first when paths come from variables.
- Honor the deletion preference recorded in `backbone.yml` `conventions.custom_rules` during first-time init.

---
> Source: [giang6283623/minimal-vibe-coding-kit](https://github.com/giang6283623/minimal-vibe-coding-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
