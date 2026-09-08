## penpal

> - Treat the Release Workflow section in `CONTRIBUTING.md` as authoritative.

# Repository Instructions

## Releases

- Treat the Release Workflow section in `CONTRIBUTING.md` as authoritative.
- Use the repository-local `penpal-release` skill for actual publication.
- Publish only when the user explicitly requests a specific version. A request
  to prepare, plan, review, or test a release does not authorize publication.
- An explicit publication request authorizes the skill to update, commit, and
  push the two package version files as documented in `CONTRIBUTING.md`.
- Never run `npm publish` locally; dispatch the repository's Release workflow.
- Never overwrite, move, or force-push an existing release tag.

---
> Source: [Aaronius/penpal](https://github.com/Aaronius/penpal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
