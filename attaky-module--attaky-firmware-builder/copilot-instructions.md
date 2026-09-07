## attaky-firmware-builder

> This repository is a skill plugin that walks a user from a firmware

# attaky-firmware-builder — agent guide

This repository is a skill plugin that walks a user from a firmware
idea (or an existing open-source firmware to port) to a flashed,
boot-verified build for Attaky modular handhelds.

## The seven skills

| Skill | Role |
|---|---|
| `firmware-brainstorm` | Rough idea → approved concept doc |
| `firmware-spec` | Concept → parts list, module combination, requirements, dev plan |
| `firmware-ui` | Optional: spec → style tokens, screen layouts, sprites, HTML preview |
| `firmware-codegen` | Spec (+ optional UI package) → firmware source (PlatformIO or ESP-IDF) |
| `firmware-build` | Source → flashable `.bin` set + build manifest, with an auto-fix loop |
| `firmware-deploy` | Flash over USB, verify boot over serial, gate milestones on real hardware |
| `firmware-port` | Alternate entry: port an existing open-source firmware to Attaky hardware |

Two lanes through the chain:

- **Idea lane**: brainstorm → spec → ui (optional) → codegen → build → deploy
- **Porting lane**: port (survey → plan → implement) → build → deploy

## Using this repo without a skill runtime

If your agent platform does not load skills natively, open
`skills/<name>/SKILL.md` and follow it verbatim — each SKILL.md is a
complete, self-contained procedure. Load the reference files it names
(relative paths inside the repo) as it instructs.

## Standing invariants (apply to every skill)

- **Hardware facts come only from `shared/hardware-catalog.md`** — the
  single authority for modules, pins, addresses, Complete Builds. Never invent
  or override a catalog fact; surface disagreements to the user.
- All user-facing artifacts live in the user's **workspace folder**
  (concept, spec/, ui/, port/, source/, build/, deploy/, references/,
  tmp/) per the conventions in `firmware-brainstorm` §7 and
  `firmware-port` §1.
- Real-hardware claims require serial/behavior evidence — "it flashed"
  is never "it works".

## Installing as native skills

See `README.md` § Install for per-platform instructions (Claude Code,
Codex, claude.ai, OpenCode, Kilo Code). Single-skill copies must use
the vendored builds from `tools/build-dist.sh` — copying a folder
straight out of `skills/` breaks its `../../shared/` links.

---
> Source: [Attaky-Module/Attaky-Firmware-Builder](https://github.com/Attaky-Module/Attaky-Firmware-Builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
