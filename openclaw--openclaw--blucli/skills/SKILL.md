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
> Converted and distributed by [TomeVault](https://tomevault.io) | [Claim this content](https://tomevault.io/claim/openclaw/openclaw)
<!-- tomevault:3.0:skill_md:2026-04-07 -->
