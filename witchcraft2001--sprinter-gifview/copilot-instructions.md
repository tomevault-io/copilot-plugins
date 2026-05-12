## sprinter-gifview

> This repository contains a Sprinter DSS GIF viewer written in Z80 assembly for `sjasmplus`.

# Repository Guidelines

## Project Structure & Module Organization

This repository contains a Sprinter DSS GIF viewer written in Z80 assembly for `sjasmplus`.

- `src/` contains program sources. `gifview.asm` is the main DSS EXE entry point; `cache_code.asm` is copied to cache/Win0 and assembled with `DISP`.
- `include/` contains shared DSS/BIOS constants and EXE header helpers.
- `tools/` contains build, package, image, and verification scripts.
- `demo/` contains sample GIF files copied into the test image (`1.gif`, `2.gif`, `3.gif`).
- `build/` and `distr/` are generated output directories and should not be committed.
- `references/` is local reverse-engineering/reference material and is ignored by Git.

## Build, Test, and Development Commands

- `make build` or `tools/build.sh`: assemble `src/gifview.asm` into `build/GIFVIEW.EXE`.
- `make package` or `tools/package.sh`: create `distr/gifview.zip` with the EXE, `readme.txt`, and demo GIFs.
- `make image` or `tools/image.sh`: create `distr/gifview.img`, a FAT12 test image populated like the `kode` reference project.
- `make clean`: remove generated build output.
- `tools/verify_disasm.sh`: verify recovered reference disassemblies when working on `references/` locally.

Required external tools: `sjasmplus`; `zip` for packaging; `mtools` (`mformat`, `mcopy`, `mmd`, `mdir`) for image creation and inspection.

## Coding Style & Naming Conventions

Use uppercase Z80 mnemonics and constants, with labels in descriptive PascalCase or clear all-caps constants. Keep DSS/BIOS calls symbolic, for example `RST Dss.Rst` and `LD C,Dss.Read`. For 16-bit function/subfunction loads, keep one instruction and use formulas such as:

```asm
LD BC,#0100 * Dss.Find.DosName + Dss.F_First
```

Use spaces for alignment, avoid tabs in new files, and keep comments short and behavior-focused.

## Testing Guidelines

At minimum, run `make build` before submitting changes. Run `make package` and `make image` when changing scripts, assets, `readme.txt`, or distribution layout. If `demo/*.gif` changes, inspect the image with `mdir -i distr/gifview.img ::/DEMO`.

## Commit & Pull Request Guidelines

This repository currently has no commit history, so use concise imperative commit messages, for example `Add GIF command-line parser` or `Implement FAT12 image packaging`. Pull requests should describe the user-visible behavior, list commands run, and note any missing manual/emulator validation.

## Agent-Specific Instructions

Do not commit generated files from `build/` or `distr/`. Do not remove or rewrite ignored `references/` content unless explicitly requested; it is local research material used to guide implementation.

## External reference sources
- You may consult the following local sibling repositories/directories for answers, platform details, and implementation ideas:
  - `/Users/dmitry/dev/zx/sprinter/sprinter_bios`
  - `/Users/dmitry/dev/zx/sprinter/sprinter_dss`
  - `/Users/dmitry/dev/zx/sprinter/sprinter_ai_doc/manual`
  - `/Users/dmitry/dev/zx/sprinter/sources/tasm_071/TASM`
  - `/Users/dmitry/dev/zx/sprinter/sources/fformat/src/fformat_v113`
  - `/Users/dmitry/dev/zx/sprinter/sources/fm/FM-SRC/FM`
  - `/Users/dmitry/dev/zx/sprinter/gfxview`
  - `/Users/dmitry/dev/zx/sprinter/flappybird`
  - `/Users/dmitry/dev/zx/sprinter/flexnavigator`
  - `/Users/dmitry/dev/zx/sprinter/sources/nupogodi`
  - `/Users/dmitry/dev/zx/sprinter/sources/2DSTUDIO`
  - `/Users/dmitry/dev/zx/sprinter/sources/DOOM2`
  - `/Users/dmitry/dev/zx/sprinter/sdcc-sprinter-sdk`
- Treat them as reference material only; this repository remains the source of truth for changes you make here.

---
> Source: [witchcraft2001/sprinter_gifview](https://github.com/witchcraft2001/sprinter_gifview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
