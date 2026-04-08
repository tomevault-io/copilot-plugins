---
name: blucli
description: BluOS CLI (blu) for discovery, playback, grouping, and volume. Use when this capability is needed.
metadata:
  author: openclaw
---

# blucli (blu)

Use `blu` to control Bluesound/NAD players.

Quick start
- `blu devices` (pick target)
- `blu --device <id> status`
- `blu play|pause|stop`
- `blu volume set 15`

Target selection (in priority order)
- `--device <id|name|alias>`
- `BLU_DEVICE`
- config default (if set)

Common tasks
- Grouping: `blu group status|add|remove`
- TuneIn search/play: `blu tunein search "query"`, `blu tunein play "query"`

Prefer `--json` for scripts. Confirm the target device before changing playback.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/openclaw)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/openclaw)
<!-- tomevault:4.0:skill_md:2026-04-08 -->
