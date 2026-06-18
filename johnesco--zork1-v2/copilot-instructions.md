## zork1-v2

> This is a **frozen published snapshot** of Zork I. New work does NOT happen here.

# Zork I — v2: Bug Fixes & Testing (FROZEN)

This is a **frozen published snapshot** of Zork I. New work does NOT happen here.

| | |
|---|---|
| Repo | `Johnesco/zork1-v2` |
| Hub registration | `zork1-v2` (`versionOf: zork1`, `versionLabel: v2 — Bug Fixes`) |
| Companion repos | `zork1` (Current), `zork1-v0` (ZIL), `zork1-v1`, `zork1-v3` |
| Golden seed | **13** (in `tests/seeds.conf`) |
| Binary | `zork1-v2.ulx` (compiled from this repo's `story.ni`) |

## What v2 is

The first version that *changes the game* rather than just translating it — but still within ZIL's intent. v2 fixes:
- Bugs from the original ZIL source (things Infocom shipped broken)
- Translation bugs introduced during the v1 port

Behavior aims to match what ZIL *meant* to do, not what Inform 7 might do differently. The testing methodology was also established at this version — deterministic walkthroughs, regtests, transcript comparison.

## Editing rules

**Default:** Don't edit this repo. Work happens in **`Johnesco/zork1`** (Current).

This repo is only edited when:
- A bug fix or enhancement must be applied at the v2 level
- The fix must then propagate **upward** to v3 and Current (never downward to v1)

If you patch this repo:
1. Edit `story.ni` here
2. Recompile: `python /c/code/ifhub/tools/pipeline.py zork1-v2 compile`
3. Run walkthrough with golden seed 13: `python /c/code/ifhub/tools/testing/run_walkthrough.py --config tests/project.conf --seed 13`
4. Commit + push (auto-deploys to `johnesco.github.io/zork1-v2/`)
5. Apply the same change to `zork1-v3` and `zork1` (Current), recompile each

For the full multi-version workflow, see `C:\code\ifhub\reference\multi-version-guide.md`.

## Project-wide context

For Zork-specific game systems, scoring, and project workflow conventions, see the main repo's `CLAUDE.md` at `C:\code\text-games\i7\zork1\CLAUDE.md`.

---
> Source: [Johnesco/zork1-v2](https://github.com/Johnesco/zork1-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
