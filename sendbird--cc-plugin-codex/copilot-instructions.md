## cc-plugin-codex

> - This is a Node.js 18+ ESM Codex plugin. Keep behavior portable across macOS, Linux, and native Windows.

# Repository guidance

- This is a Node.js 18+ ESM Codex plugin. Keep behavior portable across macOS, Linux, and native Windows.
- Fix shared behavior at the earliest common boundary, usually in `scripts/lib/`; do not add caller-specific workarounds.
- When changing a skill or runtime contract, keep `skills/`, `internal-skills/`, `scripts/`, and the matching tests in sync.
- Run `npm run check` before merging.
- For releases, update `CHANGELOG.md` and keep `package.json` and `.codex-plugin/plugin.json` versions synchronized.

---
> Source: [sendbird/cc-plugin-codex](https://github.com/sendbird/cc-plugin-codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
