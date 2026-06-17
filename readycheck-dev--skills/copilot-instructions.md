## skills

> This repository is the published Claude Code marketplace and plugin source for ReadyCheck.

# ReadyCheck Skills Repository Policy

This repository is the published Claude Code marketplace and plugin source for ReadyCheck.

## Release Policy

- Do not update `plugins/readycheck` by pushing changes directly to this repository.
- Do not bump the ReadyCheck plugin version in `.claude-plugin/marketplace.json` manually in this repository.
- The published ReadyCheck plugin and marketplace version must be updated only by the GitHub `Release` workflow in `readycheck-dev/ReadyCheck`.
- If a new ReadyCheck plugin version is needed, trigger the `Release` workflow from `readycheck-dev/ReadyCheck` on `main`.

## Reason

- `readycheck-dev/ReadyCheck` is the source-of-truth repository.
- Its release workflow packages the runtime, prepares the published plugin source, and syncs this repository in one step.
- Direct pushes here can desynchronize the marketplace from the published release assets and break plugin upgrades.

---
> Source: [readycheck-dev/skills](https://github.com/readycheck-dev/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
