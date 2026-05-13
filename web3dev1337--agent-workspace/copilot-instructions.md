## agent-workspace

> Read `CLAUDE.md` and `CODEBASE_DOCUMENTATION.md` before making changes.

# Agent Workspace Repo Instructions

Read `CLAUDE.md` and `CODEBASE_DOCUMENTATION.md` before making changes.

## Release Versioning
- `package.json` is the release version source of truth.
- Keep `src-tauri/tauri.conf.json` and `src-tauri/Cargo.toml` synced with `npm run release:sync-version`.
- Run `npm run release:check-version` before tagging or shipping a release build.
- Release tags must be `v<package.json version>`.
- `scripts/tauri/run-tauri-build.js` clears stale `bundle/` outputs and verifies that installer filenames include the expected version before CI uploads them.

---
> Source: [web3dev1337/agent-workspace](https://github.com/web3dev1337/agent-workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
