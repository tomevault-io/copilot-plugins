## mwc-tock-api

> This repo uses **CLAUDE.md** as the single shared source of truth for all agents

# AGENTS.md

This repo uses **CLAUDE.md** as the single shared source of truth for all agents
(Claude Code, Codex, Cursor). Read it before doing anything.

Non-negotiables:
1. **Privacy invariant** — only `time`, `firstName`, `lastInitial`, `status` may ever
   reach the Google Sheet, the `/waitlist.json` feed, or the board. Drop everything
   else at ingest. See CLAUDE.md > "The one rule that overrides everything".
2. **No secrets in git** — `SHARED_SECRET` in the n8n JSON is a placeholder.
3. **Human review gate** — anything touching the production n8n instance or the live
   kiosk board is proposed as a diff, never auto-deployed.

---
> Source: [Tylermeans/mwc-tock-api](https://github.com/Tylermeans/mwc-tock-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
