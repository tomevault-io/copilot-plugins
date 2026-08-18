## investor-data-room

> This repository **is** a Claude skill (also usable by Codex / Gemini skill runners).

# AGENTS.md - investor-data-room

This repository **is** a Claude skill (also usable by Codex / Gemini skill runners).

- The skill contract lives in [`SKILL.md`](SKILL.md). Read it first.
- Two modes: **BUILD** (assemble an investor packet from an `inputs/` folder) and
  **SERVE** (the embedded analyst prompt inside the generated `context.md`).
- Setup and the exact kickoff phrase: [`install.md`](install.md).
- Source-discipline rules (confidence levels, claim format): `references/evidence-ledger.md`.
- The embedded investor-facing prompt: `references/avatar-prompt.md`.

## Hard rules for any agent running this

- Whitelist-first. Nothing leaves the packet unless it passes the evidence ledger
  and is explicitly investor-safe. When in doubt, keep it home.
- Never copy raw `inputs/` (chat archives, drafts) into the output. Distill only the
  logic and the sourced facts.
- Never include internal/sensitive material (see `templates/internal-blocklist.md`).
- Never send anything. Assemble and show; the founder decides what goes out.
- Never commit `inputs/` or generated `investor-packet-*` folders (see `.gitignore`).

---
> Source: [igindin/investor-data-room](https://github.com/igindin/investor-data-room) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
