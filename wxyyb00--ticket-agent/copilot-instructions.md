## ticket-agent

> `CLAUDE.md` is this repository's canonical and complete agent instruction file.

# Codex agent entrypoint

`CLAUDE.md` is this repository's canonical and complete agent instruction file.
Read it fully before any project action; this file only supplies the minimal
Codex-discoverable routing needed to reach the canonical instructions.

## Startup Workflow

1. Confirm the project root, then read `CLAUDE.md` and `README.md` completely.
2. Run `./init.sh` before editing.
3. Read `feature_list.json`, `progress.md`, and `session-handoff.md` when present.
4. One feature at a time: work on exactly one active feature and stay inside its documented scope.

## End of Session / Completion and handoff

- Do not claim completion without running `npm test`; run `npm run build` after frontend changes and manually inspect changed UI interactions.
- Record status and verification evidence in `feature_list.json` and `progress.md`; update `session-handoff.md` for a substantial session.
- Follow every invariant and the full definition of done in `CLAUDE.md`.
- Leave a clean, restartable state where the next session can immediately run `./init.sh`.
- Do not duplicate the detailed rules here; keeping them in `CLAUDE.md` prevents the two entrypoints from drifting.

---
> Source: [wxyyb00/ticket-agent](https://github.com/wxyyb00/ticket-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
