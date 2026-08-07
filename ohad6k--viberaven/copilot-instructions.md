## viberaven-core

> VibeRaven production gate — canonical agent commands and gate loop


Run `npx -y viberaven check` (or the legacy `npx -y viberaven --agent-mode`) before deploy, auth, RLS, webhooks, or dependency changes.

Read `.viberaven/agent-tasklist.md` first, then `.viberaven/gate-result.json` and `.viberaven/mission-map.md`.
Fix one gap, then `npx -y viberaven --verify`. Before CI or production promote: `npx -y viberaven --strict`.

Gate is not clear until `gate.status === "clear"`.

Before editing these files, read `.viberaven/agent-context.md` and `.viberaven/mission-map.md`.

Full production rules: `AGENTS.md` / `CLAUDE.md` (installed by `npx -y viberaven init --agents all`).

---
> Source: [ohad6k/VibeRaven](https://github.com/ohad6k/VibeRaven) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
