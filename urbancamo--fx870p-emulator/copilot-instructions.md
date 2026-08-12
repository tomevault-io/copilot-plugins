## fx870p-emulator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev          # dev server on http://localhost:3007/fx870p-emulator/ (with /log endpoint)
npm run build        # vue-tsc type-check + vite build → dist/
npm run preview      # serve the production build locally
npm run dis          # run the CLI disassembler: npx tsx tools/dis.ts <romfile> [start] [end]
npm run compile      # compile BASIC to HD61700 machine code: npx tsx tools/compiler/compile.ts <file.bas>
npm test             # run vitest test suite (headless emulator tests)
```

Type-check via `vue-tsc -b` (run as part of `npm run build`).

ROM files are **not in the repo** — place `rom0.bin`, `rom1.bin`, `charset.bin` in `public/roms/` before running.

## Architecture Overview

Pure-browser emulator for the Casio FX-870P (Hitachi HD61700 CPU). No backend. State persisted to IndexedDB. A Vite dev-server plugin adds `POST /log` → `emulator-debug.log` for debug tracing.

### Emulator module pipeline

```
emulator.ts   rAF loop → cpuRun() [cpu.ts]
                                  → execInstr() [exec.ts]  ← 256-entry dispatch table
                                  → firePcMonitor() [def.ts]
              LCD: lcdRender() [lcd.ts] → Canvas ImageData
              I/O: onSerialTick() [port.ts], commDecTimer() [comm.ts]
```

All CPU state lives as **module-level `let` variables** in `def.ts`. Other modules import them by value and use setters (e.g. `setUa(v)`) to mutate. This is the ES module constraint workaround for mutable shared state.

### Memory map (18-bit physical address space)

| Region | Physical range  | Access      | File                        |
|--------|-----------------|-------------|-----------------------------|
| ROM0   | 0x00000–0x00BFF | 16-bit word | `rom0.bin`                  |
| ROM1   | 0x00C00–0x0FFFF | byte        | `rom1.bin`                  |
| RAM0   | 0x10000–0x1FFFF | byte        | (volatile)                  |
| ROM1b  | 0x20000–0x2FFFF | byte        | `rom1.bin` (offset 0x10000) |

**Critical**: `mr[]` in `def.ts` is only 36 bytes — the CPU register file, **not** the 64 KB address space. RAM is `memdef[2].data`.

### Address translation (`addr18` in `def.ts`)

```typescript
addr18(segment, offset):
  if (offset < ((ib << 8) & 0xC000)) segment = 0  // low addresses always ROM
  return offset + ((segment & 3) << 16)
```

- Stack (`push`/`pop`): segment = `ua >> 2`
- IX data (`ldd`/`std`): segment = `ua >> 4`
- IZ data: segment = `ua >> 6`
- Interrupt fetch: segment = 0 (forced, ignores UA)
- Normal fetch uses `delayed_ua` (UA from the previous instruction cycle)

For the stack to be in RAM, bits 3:2 of UA must be `01` (e.g. UA=0x14, 0x44, 0x54, 0xD4).

### HD61700-specific quirks

**`jr` offset encoding** — negative offsets use `0x80 - raw` (word arithmetic), NOT standard sign-extension. Raw byte 0x99 → offset = `0x80 - 0x99 = -0x19`. Both `disassemble.ts` and `tools/dis.ts` implement this correctly. **Do not change to `raw - 0x100`.**

**Flag register** — `setFlagsB(x)` sets Z_bit when `x ≠ 0` (inverted from standard CPUs). Condition code `nz` fires when Z_bit is SET (result was non-zero).

**Condition codes** in `testCC()` use `opcode[0] & 7` (lower 3 bits), not the upper bits. CC=7 = unconditional.

**`cal`/`rtn` return address** — `cal` pushes `pc - 1`; `rtn` pops and adds 1. Net result is correct return.

**`anc $1,$sz`** — reads `mr[sz]` (a general register at index `sz`), ANDs with `mr[1]`. If `sz = 0`, result is always 0 → z=1.

### Debug infrastructure

`emulator.ts` installs two monitors via `def.ts`:
- `setRamWriteMonitor(fn)` — called on every RAM write (used for boot-phase diagnostics)
- `setPcMonitor(fn)` — called before each instruction via `firePcMonitor(pc)` in `cpu.ts`

`remote-log.ts` batches log entries and POSTs to `/log` when `isRemoteLogEnabled()` is true. Toggle from the **Log** button in `CommPanel.vue`, or `window.ioDebug(true)` in the browser console.

### Debug workflow

1. `npm run dev`
2. Open browser → click **Log** (turns red) **before** clicking **Fresh Start**
3. Wait ~3 s for boot
4. Click **Log** again to flush
5. Read `emulator-debug.log`

## Reference files

| File                                  | Purpose                                                                                       |
|---------------------------------------|-----------------------------------------------------------------------------------------------|
| `reference/fx870p-rom-annotations.md` | Annotated ROM labels, call-flow, and bug analysis — **read this first** for any ROM debugging |
| `reference/fx870p-roms.md`            | Full ROM disassembly (regenerate with `npm run dis` after any disassembler fix)               |
| `reference/ROM Disassembly/*.src`     | Inline-commented source disassemblies (authoritative)                                         |
| `reference/fx870_es/`                 | Original Delphi 5 reference implementation                                                    |

## Known CPU emulation bugs fixed

| Instruction                        | Opcode range            | Bug                                                                         | Fix                                                                                 |
|------------------------------------|-------------------------|-----------------------------------------------------------------------------|-------------------------------------------------------------------------------------|
| `adwSbw` (`sbcw`/`adcw` to memory) | 0xB8–0xBF (`adwSbw_B8`) | Subtraction carry used `y > 0xFF` — always false for negative JS numbers    | `(y >>> 0) > 0xFF` (matches byte-wide `adSb_38`)                                    |
| `cpuReset`                         | —                       | Did not zero `mr[]`, `sx`/`sy`/`sz`, `flag`                                 | `mr.fill(0)` + zero all size/index registers                                        |
| `subBcd`                           | — (BCD helper)          | Used arithmetic `r - 6 - 0x10` for borrow propagation instead of bitwise OR | `((r - 6) \| ((-0x10) >>> 0)) >>> 0` (matches Delphi `(r - $06) or cardinal(-$10)`) |

The `adwSbw` carry bug caused `sbcw (iz+$sy),$6` to leave C_bit=0 even when borrow occurred, making conditional-return instructions (`rtn nc`) fire incorrectly. Symptom: `LIST` returned after CR/LF header without displaying any lines.

The `subBcd` bug caused all BCD floating-point results to have the exponent off by -10 (e.g., `1 EXE` → `0.0000000001`). The Delphi reference uses `(Result - $06) or cardinal(-$10)` which performs bitwise OR with `0xFFFFFFF0` to propagate the borrow into the upper nibble, while the TypeScript used arithmetic subtraction which produces entirely different values on underflow.

## Coding in Casio JIS Standard BASIC

- Consult the manual [index.md](./public/docs/casio-jis-basic/index.md) for clarification of Casio Basic commands, as required.
- make note of the model indicators against individual BASIC commands - not all commands are available for the FX-870P/VX-4 model variant.
- when writing code use line number spacing of 10 to allow for insertion of extra lines without having to re-number all the time.
- use the ' apostrophe character for comments.
- break code up into re-usable subroutines where required.
- subroutines by convention are located on line numbers starting with a multiple of 100.

## BASIC Compiler

TypeScript compiler that translates Casio JIS Standard BASIC into HD61700 machine code.

### Pipeline

```
BASIC source → Lexer → Parser → AST → Code Generator → Assembly text → Assembler → Binary + Listing
```

### Modules (under `tools/compiler/`)

| Module | Purpose |
|--------|---------|
| `lexer.ts` | Tokenize Casio JIS BASIC (keywords, operators, literals) |
| `parser.ts` | Recursive descent parser → typed AST |
| `ast.ts` | AST type definitions |
| `codegen.ts` | BASIC AST → annotated HD61700 assembly |
| `opcodes.ts` | HD61700 instruction encoding tables (reversed from disassembler) |
| `assembler.ts` | Two-pass assembler: label resolution + binary emission |
| `listing.ts` | 132-column listing formatter |
| `loader.ts` | Generate BASIC loader program for real hardware (MODE110-based) |
| `compile.ts` | CLI entry point |

### Usage

```bash
npx tsx tools/compiler/compile.ts program.bas
# Outputs: program.bin, program.lst, program.sym, program.loader.bas
```

### Design

Generated code calls ROM routines (Bank0) for PRINT, INPUT, FP math, string operations. The ROM call wrapper uses `PST UA,&H54` to bank-switch, matching the CosmicV4 pattern. Variables stored as 9-byte BCD floating-point in RAM. Design spec: `docs/superpowers/specs/2026-04-04-basic-compiler-design.md`.

### Known Limitations

- BCD floating-point constant encoding is simplified (uses integer load, not full BCD conversion)
- Some complex programs fail at assembly stage due to remaining addressing mode edge cases
- ROM addresses for many builtin functions (SIN, COS, etc.) are placeholders (&H0000) — need to be filled in from ROM annotations

## BASIC Cruncher

Shrinks a Casio JIS BASIC program's stored size without renumbering: strips comments (keeping empty jump targets alive), applies THEN/ELSE-GOTO and LET micro-rewrites, conservatively strips spaces, merges statements onto fewer lines, and (at `--level 2`) renames variables to single/double-char names and expands `NEXT a,b` into bare `NEXT` statements. Byte counts come from the emulator's own tokenizer, so savings are exact, not estimated.

```bash
npm run crunch -- PROGRAM.BAS [-o out.bas] [-l out.lst] [--level 1|2]
  [--keep-comments] [--no-merge] [--no-spaces-strip] [--no-rewrites]
  [--no-data-group] [--width N]
```

Outputs `PROGRAM.min.BAS` (crunched source) and `PROGRAM.crunch.lst` (132-column before/after listing with a pass-by-pass byte summary). Design spec: `docs/superpowers/specs/2026-07-21-basic-compressor-design.md`.

## Headless Emulator Debugger

Debug compiled BASIC binaries and explore the FX-870P emulator programmatically.

### Usage

```bash
npm run debug -- run <binary.bin>       # run with exit reason + LCD
npm run debug -- trace <binary.bin>     # log every instruction
npm run debug -- step <binary.bin>      # interactive single-step
```

Key flags: `--break 0xADDR`, `--watch 0xADDR:w`, `--dump-regs`, `--max-cycles N`, `--boot` (slow but fresh state), `--raw` (no boot).

Library usage (from tests):
```typescript
import { EmulatorSession } from './tools/emu-debugger/session.js';
const sess = new EmulatorSession({ mode: 'snapshot' });
sess.loadBinary(0x0000, bytes);
sess.setEntry(0x0000);
const result = sess.run({ maxCycles: 1_000_000 });
console.log(result.reason, sess.getLcd().rows);
```

Snapshots auto-generated on first run; stored in `tools/emu-debugger/snapshots/` (gitignored). Design spec: `docs/superpowers/specs/2026-04-05-emu-debugger-design.md`.

## Coding of The Sorcerers Cave

Sorcerers Cave is a very ambitious Casio BASIC. Refer to the following documentation when unsure of how to implement bug fixes or enhancements:

Rules (these apply to the board game version from which this is adapted for solo play):  reference/sorcerers-cave/sorcerers-cave-rules.md

Documentation of the program (although this by necessity will be someone out of date) is rooted at: public/docs/sorcerers-cave/index.md - use this documentation to help code.

---
> Source: [urbancamo/fx870p-emulator](https://github.com/urbancamo/fx870p-emulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
