## agents-workspace

> > **This file is not consumer boot policy.** It guides agents working **in** the agents-workspace distribution repo. For the policy that consumers copy into their projects, see [`templates/AGENTS.md`](templates/AGENTS.md).

# AGENTS.md (meta — this repository only)

> **This file is not consumer boot policy.** It guides agents working **in** the agents-workspace distribution repo. For the policy that consumers copy into their projects, see [`templates/AGENTS.md`](templates/AGENTS.md).

## What this repository is

**agents-workspace** is a **template distribution**: portable skills and a workspace boot-policy template that people install or copy into *their* projects.

| Path | Role |
|---|---|
| `templates/skills/` | Skill source of truth — install globally via `./tools/install.sh` |
| `templates/AGENTS.md` | Boot-policy **template** — install globally via `./tools/install.sh` (marker-upsert) |
| This file (`./AGENTS.md`) | **Meta** guidance for developing and maintaining the distribution |

`templates/` is **not** the live operational contract until installed. Do not treat `templates/AGENTS.md` as the session boot policy for someone else's project while you work here.

## Editing source of truth

- Change skills under `templates/skills/<name>/` — then remind the user to run `./tools/install.sh` (never run it automatically).
- Change consumer boot policy in `templates/AGENTS.md` — then remind the user to re-run `./tools/install.sh` (upserts the managed marker block into each tool's global instruction file).
- Do not edit installed copies under `~/.config/opencode/skills/` or global boot-policy files (or platform equivalents).

## Working in this repo

This distribution repo maintains its own `wiki/` for knowledge about the product itself. Prefer coordination via `orchestrate` for multi-step or multi-domain work on the distribution (same convention as consumers), but **load consumer boot semantics from `templates/AGENTS.md` only when editing that template** — not as your default policy for every task here.

Full consumer operating mode (The One Rule, sticky session types, flow): [`templates/AGENTS.md`](templates/AGENTS.md).

---
> Source: [wcgomes/agents-workspace](https://github.com/wcgomes/agents-workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
