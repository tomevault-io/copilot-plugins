## kj-supercollider-experiments

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Always load the `using-supercollider` skill when working with this project.

## Project

SuperCollider experiments for generative ambient music composition on macOS.

## Running Code

```bash
# Run a .scd script headless from the terminal
echo " " | /Applications/SuperCollider.app/Contents/MacOS/sclang <file>.scd

# Stop
pkill sclang

# For interactive work, use the SuperCollider IDE — it provides server meters,
# scope, and Cmd+. to stop all sound. Load files in the IDE with:
#   this.executeFile("/Users/kdj/work/sc/<file>.scd".standardizePath);
# Scripts with s.waitForBoot wrappers need the boilerplate stripped for IDE use.
```

## Conventions

- Files use `.scd` extension (SuperCollider document).
- SynthDefs must use `.add` (not `.send` or `.load`) so Pbind can map argument names.
- SynthDefs intended for Pbind must include `gate` arg with `doneAction: Done.freeSelf`.
- Use `Pmono` for drones and continuous textures; `Pbind` for polyphonic/discrete events.
- Use `Rand`/`ExpRand` UGens for per-synth randomization, not `rrand()` (which evaluates once at compile time).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kristopherjohnson)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kristopherjohnson)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
