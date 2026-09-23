## typevoice

> Read `docs/HANDBOOK.md` first: architecture, build, debug hooks, licensing (Dodo), releasing

# TypeVoice — notes for Claude

Read `docs/HANDBOOK.md` first: architecture, build, debug hooks, licensing (Dodo), releasing
(Sparkle, notarization, `make release` / `make publish`), site deploy, credentials, decisions.

Rules that matter here:
- Never relaunch the app while a dictation may be in flight (check the log's last line).
- Never paste or print API keys; use `~/bin/dodoapi` (keychain-backed) for Dodo.
- Verify with screenshots/measurements, not claims; the owner's taste is calm and minimal.
- Keep `CHANGELOG.md` in the user's words; it becomes the in-app release notes.

---
> Source: [warplabshq/typevoice](https://github.com/warplabshq/typevoice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
