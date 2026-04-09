## super-duper-fishstick

> Read AGENTS.md first. It has the full architecture and constraints.

Read AGENTS.md first. It has the full architecture and constraints.

This is Hermes — a Gmail AI agent built on Google Apps Script + Gemini.

Before editing ANY file, read it first with `cat`. Files change between sessions.

Critical constraints:
- Google Apps Script V8 runtime — no npm, no async/await, no imports/require
- UrlFetchApp.fetch() instead of fetch()
- Logger.log() instead of console.log()
- 6-minute execution limit per trigger
- All source in src/, deployed via `clasp push` or `./deploy.sh`
- schema.md is read-only (human-edited preferences)
- wiki/ is LLM-authored compiled knowledge (don't manually edit)

After changes: run ./deploy.sh to push to Apps Script.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Snapphil)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Snapphil)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
