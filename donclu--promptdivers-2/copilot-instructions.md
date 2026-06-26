## promptdivers-2

> Promptdivers pack rule (dogfood). Keeps sessions governed: read context first, avoid invented facts, log decisions, and route work via squads/stratagems.


## Promptdivers rule — governed operation

### First contact

- Read `AGENTS.md` before making changes.
- If `PROJECT_LOG.md` exists, read the latest session entry and `HANDOFF_JSON`.
- If `GALACTIC_WAR_MAP.md` exists, do the pre-drop planet check (`protocols/pre-drop.md`).

### No invented canon / no invented APIs

- Treat *Helldivers*-style language in this repo as **metaphor**. Do not assert game canon or mechanics as fact.
- In code or docs: do not invent file paths, APIs, tools, or commands. If unsure, search the repo and cite evidence.

### Scope + evidence discipline

- Keep scope locked to the user’s objective; log any scope expansion as `[PENDING SCOPE]` in `PROJECT_LOG.md`.
- Prefer verifiable actions (diffs, tests, commands) over narrative. If you claim “verified”, show what was verified.

### Token economy (default)

- Use the lightest coordination that unblocks progress:
  - **SOLO**: ≤5 files, clear next step.
  - **RNF (Reinforce)**: needs parallel drafting/exploration.
  - **SOS**: blocked and needs a human decision/access/intent.
  - **ESCALATE**: high-risk policy/security/scope-contract change.

### End of phase

- When objectives exist, close with a Pelican debrief (`protocols/mission-debrief.md`) and update `PROJECT_LOG.md`.

---
> Source: [donclu/Promptdivers-2](https://github.com/donclu/Promptdivers-2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
