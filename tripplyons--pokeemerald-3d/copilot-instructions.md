## pokeemerald-3d

> This repo is a pokeemerald decompilation with an in-progress WebAssembly port.

# Agent Instructions

This repo is a pokeemerald decompilation with an in-progress WebAssembly port.
The primary goal is to run the game in the browser while preserving the original
source tree and behavior as much as possible.

## Project Direction

- Prefer using the original pokeemerald source, data, and asset pipeline
  directly.
- Avoid modifying original gameplay/source/data files when a WASM-only wrapper,
  generated file, browser shim, or build rule can solve the problem.
- When a change to original source is necessary for the WASM port, keep it as
  small as possible and guard behavioral differences with `#if WASM` or
  `#ifdef WASM`.
- Preserve the original gameplay behavior unless the user explicitly asks to
  change it. WebAssembly is the repository's only supported build target.

## WASM Port Layout

- `make wasm` builds `build/wasm/pokeemerald.wasm`.
- `make serve-wasm` builds the WASM target and serves the browser frontend.
- Browser runtime code lives in `web/`.
- WASM include shims live in `include/wasm/`.
- WASM generators and conversion tools live under `tools/`, for example
  `generate_wasm_assets.py`, `generate_wasm_text.py`, and `wasm_asm_data.py`.
- WASM-specific source/data that cannot come directly from upstream data should
  live under a clearly named WASM path such as `data/wasm/`.

## Implementation Rules

- Prefer adding WASM adapters at the boundary:
  - build rules in `Makefile`
  - browser glue in `web/`
  - compatibility headers in `include/wasm/`
  - generated C/asm/data under `build/wasm/`
  - source generators in `tools/`
- If original C files need WASM-specific behavior, isolate the exact difference
  with `#if WASM` near the affected code.
- Avoid broad rewrites of game logic to make the browser port easier.
- Avoid duplicating large source/data files. Generate translated forms when
  possible.
- Keep generated outputs out of git. Check `.gitignore` before committing if new
  build artifacts, screenshots, wasm files, or temporary data are created.
- The root `.gitignore` already ignores `build/`, `*.o`, `*.elf`, `*.gba`,
  `*.map`, `*.sym`, and most generated asset extensions. Add focused ignore
  rules if new generated artifacts fall outside those patterns.
- For every crash or gameplay bug, first identify the root cause and fix the
  shared conversion/runtime/build layer that caused it. Do not patch individual
  maps, events, cutscenes, battles, moves, or assets unless the root cause truly
  is isolated to that one piece of original content.
- Prefer fixes that apply automatically to whole classes of equivalent native
  data, commands, assets, or runtime paths. When a bug presents in one example,
  check for the underlying pattern and verify at least one representative path
  instead of hardcoding that example.
- After each verified change, commit the completed work before starting the next
  fix or feature, unless the user explicitly asks to keep changes uncommitted.

## Reproducible WASM Replay

Use `tools/wasm_replay.mjs` to run the normal `web/` frontend in a headless
Chrome/Chromium instance without the animation-frame cap and save deterministic
canvas screenshots, browser console output, and errors.

```sh
node tools/wasm_replay.mjs path/to/events.txt
```

The tool runs `make wasm`, starts `web/server.mjs`, opens `/?automate=1`, applies
input events at exact emulated game frame numbers, recreates the fixed
`wasm-replay-output/` folder, and writes:

- `screenshots/*.png` for requested screenshot frames
- `console.log` for server, browser, and page console messages
- `errors.log` for CLI or page exceptions
- `events.json` and `summary.json` for run metadata

Pass `--no-build` to reuse an existing `build/wasm/pokeemerald.wasm`. Set
`CHROME_BIN=/path/to/chrome` if Chrome/Chromium is not in a standard location.
The replay event file format is line oriented. Frame numbers are calls to
`WasmRunFrame()` / emulated game frames, not browser display frames:

```text
0 screenshot boot
120 button start on
124 button start off
240 screenshot title-screen
```

Valid buttons are `a`, `b`, `select`, `start`, `right`, `left`, `up`, `down`,
`r`, and `l`. `#` starts a comment. Keep reusable replay inputs in
`tools/wasm_replays/` and add screenshot events at meaningful route checkpoints
so future runs show where the automation is.

## Build And Verification

For WASM changes, run:

```sh
make wasm
```

For browser/runtime changes, also run:

```sh
make serve-wasm
```

Then open the served page and verify the game reaches the intended flow.

Use the narrowest verification that gives real confidence, and report any tests
or builds that could not be run.

## Commit Guidance

- Follow the style of recent commits. Current WASM commits use short imperative
  subjects such as `Fix wasm window rendering` and `Add wasm FPS counter`.
- If asked to commit, inspect staged files before committing.
- Before committing, update `.gitignore` files if any generated or local-only
  artifacts should not be committed.
- Do not revert or overwrite uncommitted user changes unless the user explicitly
  asks.

## When Stuck

If you get stuck, spawn a subagent to answer the specific question blocking the
work. Keep the question narrow and include the relevant files, commands, and
failure output.

---
> Source: [tripplyons/pokeemerald-3d](https://github.com/tripplyons/pokeemerald-3d) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
