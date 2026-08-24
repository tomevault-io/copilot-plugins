## cyrius-doom

> > **Core rule**: this file is **preferences, process, and procedures** — durable rules that change rarely. Volatile state (current version, binary sizes, dep pins, in-flight slots, gates, consumers, recent releases) lives in [`docs/development/state.md`](docs/development/state.md), refreshed every release. Historical release narrative lives in [`docs/development/completed-phases.md`](docs/development/completed-phases.md). Do not inline state here — inlined state rots within a minor.

# Cyrius DOOM — Claude Code Instructions

> **Core rule**: this file is **preferences, process, and procedures** — durable rules that change rarely. Volatile state (current version, binary sizes, dep pins, in-flight slots, gates, consumers, recent releases) lives in [`docs/development/state.md`](docs/development/state.md), refreshed every release. Historical release narrative lives in [`docs/development/completed-phases.md`](docs/development/completed-phases.md). Do not inline state here — inlined state rots within a minor.

---

## Project Identity

**cyrius-doom** (homage to id Software's DOOM, 1993) — Clean-room DOOM engine in Cyrius. Direct framebuffer, no libc, no SDL, kernel syscalls only.

- **Type**: Standalone game binary / kernel demo
- **License**: GPL-3.0-only (clean-room implementation from documented specs)
- **Language**: Cyrius (toolchain pinned in `cyrius.cyml [package].cyrius` — `cycc 6.1.29` at the time of writing; canonical pin is the file)
- **Version**: `VERSION` at project root is the source of truth (referenced via `version = "${file:VERSION}"` in `cyrius.cyml`). Do not inline the number here.
- **Genesis repo**: [agnosticos](https://github.com/MacCracken/agnosticos)
- **Standards**: [First-Party Standards](https://github.com/MacCracken/agnosticos/blob/main/docs/development/planning/first-party-standards.md) · [First-Party Documentation](https://github.com/MacCracken/agnosticos/blob/main/docs/development/planning/first-party-documentation.md)
- **Philosophy**: [AGNOS Philosophy](https://github.com/MacCracken/agnosticos/blob/main/docs/philosophy.md)

## Goal

Own a clean-room DOOM engine in Cyrius — runnable on /dev/fb0, on AGNOS, and eventually on bare metal. No id Software code copied; everything from documented specs. Proves Cyrius can drive a real-time renderer at original-DOOM fidelity without libc, FPU, or external runtime.

## Current State

> Volatile state lives in [`docs/development/state.md`](docs/development/state.md) — current version, binary sizes, dep pins (bsp / vani tags), in-flight slot status, last-green gates, known-issue workarounds. Refreshed every release.
>
> Historical release narrative lives in [`docs/development/completed-phases.md`](docs/development/completed-phases.md). Per-release detail lives in [`CHANGELOG.md`](CHANGELOG.md).
>
> Forward-facing slots live in [`docs/development/roadmap.md`](docs/development/roadmap.md).

## Scaffolding

Project was scaffolded manually; subsequent modernization passes match the patra/vani/sakshi/mihi convention (single `cyrius.cyml`, `${file:VERSION}` template, patra-style CI installer). Do not hand-roll new structure if `cyrius init` covers it — fix the tool, then re-propagate.

## Quick Start

```sh
# Build (requires the toolchain pinned in cyrius.cyml)
cyrius build src/main.cyr build/doom

# Release build: NOPs dead functions in-place (used by release.yml)
CYRIUS_DCE=1 cyrius build src/main.cyr build/doom

# Run (requires DOOM1.WAD — see scripts/get-wad.sh)
./build/doom wad/DOOM1.WAD              # interactive (/dev/fb0 or GTK bridge)
./build/doom wad/DOOM1.WAD --ppm        # game screenshot mode (headless)
./build/doom wad/DOOM1.WAD --ppm-menu   # menu screenshots
./build/doom wad/DOOM1.WAD E1M3 --ppm   # specific map

# Download shareware WAD (one-time, not in repo)
sh scripts/get-wad.sh wad

# Test
cyrius test tests/doom.tcyr                                  # WAD-free subset
cyrius build tests/doom.tcyr build/test_doom && \
  ./build/test_doom wad/DOOM1.WAD                            # full suite

# Fuzz
cyrius build fuzz/fuzz_fixed.cyr build/fuzz_fixed && ./build/fuzz_fixed
cyrius build fuzz/fuzz_wad.cyr   build/fuzz_wad   && ./build/fuzz_wad

# Bench (also appends a row to bench-history.csv)
sh scripts/bench-history.sh wad/DOOM1.WAD

# One-shot
sh scripts/run.sh
```

## Architecture (durable — module map)

```
src/
  main.cyr        — entry, game loop (35Hz tick), --ppm screenshot mode
  fixed.cyr       — 16.16 fixed-point math, native `>>>` arithmetic right shift
  tables.cyr      — 1024-entry sine table (Bhaskara I), atan2, trig wrappers
  wad.cyr         — WAD parser (IWAD/PWAD, directory, lump read/cache)
  framebuf.cyr    — 320x200 palette-indexed framebuffer, PPM output
  map.cyr         — vertices, linedefs, sidedefs, sectors, segs, subsectors, BSP nodes, things
  texture.cyr     — wall texture compositing from patches, flat cache, patch LRU cache
  render.cyr      — BSP traversal, textured wall columns, COLORMAP lighting, visplane spans, sky
  sprite.cyr      — thing sprites: distance sort, scale, clip to walls, sector lighting
  input.cyr       — terminal raw mode, WASD + arrows, bitmask action flags
  player.cyr      — movement, wall sliding collision, step height, ceiling check
  tick.cyr        — 35Hz timer via clock_gettime + nanosleep
  things.cyr      — monster AI state machine, item pickups, damage
  status.cyr      — HUD: bitmap font, health/ammo/armor/face/keys
  sound.cyr       — PC speaker tone queue via ioctl
  audio.cyr       — WAD SFX loading + ALSA playback via vani
  music.cyr       — MUS (D_*) parser + sequencer + wavetable synth, mixed into audio out
  doors.cyr       — door / lift sector animation, tagged sectors, walk triggers
  automap.cyr     — 2D overhead map (TAB toggle, Bresenham lines)
  level.cyr       — episode/map tracking, exit lines, level stats
  menu.cyr        — WAD-native title screen (TITLEPIC), main menu, skill select

  platform/                    — desktop-display backends (v0.33.0), all #ifndef CYRIUS_TARGET_AGNOS
    wayland/wire.cyr    — Wayland wire-protocol codec (pure, no syscalls)
    wayland/client.cyr  — AF_UNIX connect + registry/bind + xdg dispatch + SCM_RIGHTS fd-passing + key ring
    wayland/shm.cyr     — double-buffered wl_shm present (memfd/mmap, two-buffer ping-pong pool)
    window.cyr          — the win_* seam (present/poll/resize/key/close); fb0/AGNOS/Wayland behind one contract
```

The Wayland backend is sovereign (raw wl protocol via syscalls — no libwayland, no new deps) and selected at
runtime via `present_mode` (PM_FB0 / PM_WAYLAND / PM_PPM); `framebuf_flip`/`input_poll` branch to it on Linux,
fb0/AGNOS/`--ppm` byte-identical. Any build that includes `src/framebuf.cyr` on Linux (tests, fuzz_weapon,
`benches/doom.bcyr`) must also include the four `platform/` files first (framebuf references `win_*`).

Dep pins (bsp / vani versions) and module/lib counts live in [`state.md`](docs/development/state.md). What composes what is durable; the version numbers are not.

## Key Principles

- **Correctness is the optimum sovereignty** — wrong code doesn't own anything; bugs own you. Verify before claiming.
- **22 ms frame budget @ 35 Hz** — never skip benchmarks on changes to the render path. Measure before and after.
- **Fuzz early** — `fuzz/fuzz_wad.cyr` + `fuzz/fuzz_fixed.cyr` have found bugs unit tests missed. Run before claiming robustness.
- **`>>>` everywhere on signed shifts** — Cyrius `>>` is LOGICAL, so a signed right-shift must use
  the native arithmetic shift `>>>` (floors negatives, = C's signed `>>`). *Changed at v0.34.10*: all
  96 sites migrated off the `asr()` helper, which cost a function call per shift — `fixed_mul`, which
  is literally one multiply plus that shift, went **6 ns → 4 ns** and the spawn frame **−12%**.
  `asr()` still exists in the vendored `lib/bsp.cyr` (bsp's own internals use it) and remains
  semantically identical; `tests/regression_asr.tcyr` asserts the two agree, because the whole
  engine's negative-coordinate math now rests on `>>>` emitting SAR.
  **Gate before ever changing this**: `>>>` was const-fold-UNGUARDED in cyrius 6.4.46–6.4.73; only
  6.4.74+ is safe. Do not migrate shift code on an older pin.
- **Lazy init guards** — `if (ptr == 0) { ptr = alloc(N); }` — prevents double-alloc and null deref. Pattern lives in framebuf / texture / masked-segs.
- **Enum for constants** — saves `gvar_toks` slots (cycc limit: 4,096 initialized globals). Use `var` only for mutable state.
- **Initialized-globals counting rule** — *corrected 2026-07-25 (cycc 6.4.74); the old wording here
  was wrong in both directions.* As of 6.4.74 **any** constant-foldable non-zero integer expression
  at module scope takes the static-init fast path (not just a bare literal), and enum members are
  const-folded — neither consumes a `gvar_toks` slot. But **`var x = 0;` DOES consume one**: a folded
  zero deliberately falls through to the runtime-store path, and **263 of doom's 288 module-scope
  `var NAME = …` are `= 0`**. A shadowing redefinition never skips. Don't reason from the rule —
  **measure with `CYRIUS_STATS=1`**; 6.4.73/.75 also recalibrated the `fn_table` (now /32768) and
  `code_size` denominators, so utilization numbers from before then are not comparable. See the
  cyrius guide's **Global Initializers** section (`docs/guides/cyrius-guide.md` in the cyrius repo).
- **Patch cache** — `pcache_get()` eliminates WAD I/O during rendering (200× speedup). Don't bypass it.
- **Sakshi tracing** — all error paths use `sakshi_error / sakshi_warn / sakshi_info` — structured timestamped logging. Don't `file_write(2, ...)` raw.
- **Clean-room implementation** — read [Black Book](https://fabiensanglard.net/gebbdoom/) + [Unofficial Specs](https://doomwiki.org/wiki/WAD) before implementing. Never copy from [id Software DOOM source](https://github.com/id-Software/DOOM) (GPL-2.0; read for understanding only).

## Rules (Hard Constraints)

- **Do not commit or push** — the user handles all git operations
- **NEVER use `gh` CLI** — use `curl` to GitHub API only
- Do not include WAD files in the repo (gitignored)
- Do not copy id Software source — clean-room from documented specs only
- Do not use floating point — all math is 16.16 fixed-point
- Do not use bare `>>` on signed values — use `>>>` (native arithmetic shift; `asr()` is the retired helper)
- Do not skip fuzz testing before claiming robustness
- Do not allocate `var buf[N]` for buffers > 100 bytes — use `alloc()` (cycc 256 KB output limit)
- Do not hardcode toolchain versions in CI YAML — `cyrius = "X.Y.Z"` in `cyrius.cyml` is the only source of truth
- Do not inline volatile state in CLAUDE.md — it goes in `docs/development/state.md`

## Process

### P(-1): Research (before implementing any module)

1. Read the relevant chapter in DOOM Black Book.
2. Read the Unofficial DOOM Specs for data-format details.
3. Check `vidya/content/cyrius/field_notes.toml` for Cyrius-specific gotchas.
4. Check `vidya/content/cyrius/language.toml` for compiler constraints.
5. **Security research** — for the feature area:
   a. Search known CVEs (DOOM / PrBoom / Chocolate Doom / ZDoom / GZDoom).
   b. Search exploit classes (buffer overflow, integer overflow, DoS, ACE).
   c. Search malicious-WAD / savegame / network / DEHACKED attack vectors.
   d. Check `docs/audit/` for prior findings that apply.
   e. Write findings to `docs/audit/{date}-{topic}.md`.
   f. Add fix items to roadmap as next-version security tasks.
   g. Fix before shipping — no new attack surface without validation.
6. Document findings as comments in the source file header.

### Work Loop (continuous)

1. **Work phase** — feature, fix, optimize.
2. **Build check** — `cyrius build src/main.cyr build/doom`.
3. **Test** — `cyrius test tests/doom.tcyr` (WAD-free) and `./build/test_doom wad/DOOM1.WAD` (full).
4. **PPM smoke** — `./build/doom wad/DOOM1.WAD --ppm`; verify map summary matches state.md and PPMs are 192,015 B each.
5. **Fuzz** — `fuzz_wad` + `fuzz_fixed` if the change touches parser or fixed-point.
6. **Bench** — `sh scripts/bench-history.sh` for any change to the render path; verify variance-level deltas only unless a perf claim is being made.
7. **Lockfile** — `cyrius deps` (resolves `./lib/` from the pinned stdlib + `[deps.*]` git overrides and writes the lock) then `cyrius deps --verify`. `./lib/` is a **gitignored build artifact**, so the lock hashes whatever the pinned toolchain populates — the entry count tracks the pinned stdlib snapshot + the git-dep set (current count lives in [`state.md`](docs/development/state.md); it moved at 0.28.4, 0.31.0, and 0.32.0, so don't inline it here). Bumping the toolchain pin (or adding/removing a git dep) changes the resolve, so **regenerate + commit the lock in the same change** (`rm -rf lib && cyrius deps` for a deterministic resolve — the warm-cache writer can under-count when stale `./lib/` files linger).
8. **Documentation** — CHANGELOG entry, state.md refresh, roadmap forward-list update, doc-health row touched if a doc was edited.
9. **Version check** — `VERSION` (single source of truth), `src/main.cyr` banner, CHANGELOG header all in sync.

### Closeout Pass (before every minor bump)

1. Full test suite — **every** `tests/*.tcyr` passes, not just `doom.tcyr`. Naming one file is how
   `regression_asr.tcyr` sat red for two weeks after 0.33.6 (CI never ran it); CI globs the
   directory as of 0.34.5. Live counts are in [`state.md`](docs/development/state.md), not here.
2. Bench baseline — `bench-history.sh`; compare against prior closeout.
3. Dead-code audit — `CYRIUS_DCE=1` build; record NOP-sled size in CHANGELOG.
4. Refactor pass — consolidate the minor's additions where parallel codepaths accreted.
5. Code review pass — walk diffs end-to-end for missed guards, ABI leaks, off-by-ones, silently-ignored errors.
6. Cleanup sweep — stale comments, dead branches, unused includes.
7. Security re-scan — quick grep for new `sys_system`, unchecked writes, unsanitized input, buffer-size mismatches.
8. Doc sync — CHANGELOG, roadmap (forward), state.md, completed-phases.md (move shipped row in), doc-health.md (touch affected rows).
9. Version verify — `VERSION`, `cyrius.cyml`, CHANGELOG header, intended git tag all match.
10. Full build from clean — `rm -rf build && cyrius deps && CYRIUS_DCE=1 cyrius build` passes clean.

### Task Sizing

- **Low/Medium**: batch freely — multiple items per cycle.
- **Large**: small bites — break into sub-tasks, verify each.
- **If unsure**: treat as large. Research via vidya first, then externally.

### Refactoring

- Refactor when the code tells you to — duplication, unclear boundaries, measured bottlenecks.
- Never refactor speculatively. Wait for the third instance.
- Every refactor must pass the same build + test + fuzz + bench gates.

## Cyrius Conventions

- **16.16 fixed-point math** — no FPU; `fixed_mul` / `fixed_div` / `>>>` are the workhorses.
- **No libc** — direct syscalls via `lib/io.cyr`.
- **Heap-allocated large buffers** — `alloc()` for anything > 100 bytes; `var buf[N]` bloats the binary.
- **Enum-packed constants** — see `MapMax` / `MapSize` / `MapLineFlag` / `Fixed` / `Angle` / `ViewConst` / `WeaponConst` for the pattern.
- **35 Hz game tick** — matches original DOOM timing.
- **320×200 palette-indexed** — 256 colors from WAD PLAYPAL lump.
- **Multi-return tuples** — `var tx, ty = render_transform_vertex(wx, wy);` — supported since Cyrius 3.7.2.
- **Switch / case** — case labels require literal integers, not enum identifiers.
- All struct-field access via `load64` / `store64` with offsets (8-byte field convention).
- `cycc` accepts `: i64` return-type annotations as parse-only metadata (v5.11.x). Every public fn in `src/*.cyr` carries one as of v0.27.2.

## CI / Release

- **Toolchain pin**: `cyrius = "X.Y.Z"` in `cyrius.cyml [package]`. No separate `.cyrius-toolchain`. CI + release both read this; no hardcoded versions in YAML.
- **Patra-style installer**: pre-flight HTTP check on the cyrius release asset; version-pinned install layout (`~/.cyrius/versions/$V/{bin,lib}/`).
- **`cyrius.lock`**: committed supply-chain anchor — the resolver's own `cyrius deps` output; the entry count tracks the pinned toolchain's `lib/` snapshot + the git-dep set (current count in [`state.md`](docs/development/state.md) — do not inline it here). `./lib/` is gitignored, so the lock hashes the pinned stdlib + git-override deps. CI checks the lock is present, runs `cyrius deps` (resolves `./lib/` + rewrites the lock from the pinned toolchain), then `cyrius deps --verify` as the unconditional gate. The earlier cycc 6.0.1 lockfile-writer regression (empty lock → `sha256sum` hand-population + guarded verify) was resolved in 0.27.5 by the 6.0.29 pin.
- **Dead-code elimination**: release workflow runs `CYRIUS_DCE=1`. Binary size tracked.
- **CI test job**: the WAD-free subset of **every** `tests/*.tcyr` (glob, not a named file — see the
  Closeout note above). The WAD-gated assertions need a WAD path passed to the built binary and are
  exercised locally, not in CI, by design. Live counts live in `state.md`.
- **Bench history**: `bench-history.csv` appended via `scripts/bench-history.sh`; compare row-over-row.

## References

These are the primary sources for clean-room implementation. Read before implementing.

- **[Game Engine Black Book: DOOM](https://fabiensanglard.net/gebbdoom/)** — Fabien Sanglard's engine analysis (rendering pipeline, BSP, visplanes, sprites)
- **[Unofficial DOOM Specs](https://doomwiki.org/wiki/WAD)** — WAD format, lump types, map data structures
- **[DOOM Source Code](https://github.com/id-Software/DOOM)** — GPL-2.0 reference (DO NOT copy — read for understanding only)
- **[DOOM1.WAD (shareware)](https://github.com/nneonneo/universal-doom)** — test WAD, downloaded by `scripts/get-wad.sh`
- **vidya** — `content/cyrius/field_notes.toml` documents Cyrius-specific lessons from this build

## Docs

- [`docs/architecture/`](docs/architecture/) — non-obvious invariants & module-level layout
- [`docs/audit/`](docs/audit/) — security audit reports (`YYYY-MM-DD-*.md`)
- [`docs/proposals/`](docs/proposals/) — pre-ADR design drafts (`archive/` for shipped/closed)
- [`docs/sources.md`](docs/sources.md) — citations for math / engine reference material
- [`docs/development/roadmap.md`](docs/development/roadmap.md) — **forward-facing slots only**
- [`docs/development/state.md`](docs/development/state.md) — **live state snapshot, refreshed every release**
- [`docs/development/completed-phases.md`](docs/development/completed-phases.md) — chronological shipped-version index
- [`docs/doc-health.md`](docs/doc-health.md) — fresh / stale / archive ledger across the whole doc tree (lives at `docs/` root per first-party-documentation convention)
- [`CHANGELOG.md`](CHANGELOG.md) — Keep a Changelog format; per-release detail

New non-obvious quirks and constraints land in `docs/architecture/` as numbered items (`NNN-kebab-case.md`) once that subdirectory earns its second entry. New decisions land in `docs/adr/` once that subdirectory is scaffolded. Never renumber either series.

Full doc-tree convention: [first-party-documentation.md](https://github.com/MacCracken/agnosticos/blob/main/docs/development/planning/first-party-documentation.md).

## CHANGELOG format

Follow [Keep a Changelog](https://keepachangelog.com/). Performance claims **must** include benchmark numbers (frame time, binary size). Every version bump updates `VERSION` (single source of truth — `cyrius.cyml` resolves it via `${file:VERSION}`) + `src/main.cyr` banner. State.md and completed-phases.md update in the same commit.

---
> Source: [MacCracken/cyrius-doom](https://github.com/MacCracken/cyrius-doom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
