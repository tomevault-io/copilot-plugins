## x402-wallet-for-claude-desktop

> When creating a new release tag (e.g. `v0.1.5`), always update the `version` field in **all three files** to match:

# CLAUDE.md

## Versioning

When creating a new release tag (e.g. `v0.1.5`), always update the `version` field in **all three files** to match:

1. `package.json` — npm version
2. `manifest.json` — `.mcpb` installer reads the version from here
3. Git tag (e.g. `v0.1.5`)

All three must stay in sync. If any is missed the installer will show a stale version.

---
> Source: [402md/x402-wallet-for-claude-desktop](https://github.com/402md/x402-wallet-for-claude-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
