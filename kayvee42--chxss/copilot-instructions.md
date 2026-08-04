## chxss

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About

Norman Pollock's **40H** chess tool suite — 92 single-purpose command-line utilities for processing PGN (game) and EPD (position) files. "40H" is hex for 64 (squares on a chessboard).

- **Author**: Norman Pollock (rc1242@yahoo.com)
- **Upstream mirrors**: https://nk-qy.info/40h/ | https://40hchess.epizy.com/
- **Downloaded**: 2026-07-17
- **Status**: Decompiled, exhaustively reviewed, design complete. Ready for Rust port.

## Repository layout

```
src/                — Rust source code
tests/              — Integration tests
docs/               — User documentation
Cargo.toml          — Crate manifest
CLAUDE.md           — This file
40H-ref/            — Reference material (gitignored)
  Java/             — Original .class files
  Java-src/         — CFR-decompiled Java source (90/92 files, 28K lines)
  40H-EPD-2026.7z   — Original EPD tools (.exe + .cmd wrappers + readme-EPD.txt)
  40H-PGN-2026.7z   — Original PGN tools (.exe + readme-PGN.txt)
  40H-Java.7z       — Original Java .class files (archive)
  DESIGN.md         — Rust port design: architecture, decisions, module layout, phasing
  REVIEW.md         — Exhaustive catalog of all 92 tools from decompiled source
  ARCHITECTURE.md   — Architecture notes
  readmy            — KV's original notes on intent
```

## Running the original Java tools

```bash
cd 40H-ref/Java
java eloList input.pgn output.txt
```

Extract the readme docs (full usage instructions per tool):
```bash
7z x -so 40H-ref/40H-PGN-2026.7z 40H-PGN-2026/readme-PGN.txt | less
7z x -so 40H-ref/40H-EPD-2026.7z 40H-EPD-2026/readme-EPD.txt | less
```

## Decompiled source

CFR 0.152 was used to decompile all 92 `.class` files → `40H-ref/Java-src/`. Two files failed (eco500, ecoSplit) — both are simple file-splitters.

Key characteristics of the original code:
- Every tool is a standalone class with one `main()` method, zero shared code
- EOF detected by catching `NullPointerException` on `readLine()`
- Fixed-size `String[2000]` game buffers, crash on larger games
- Hardcoded output filenames (`out5.pgn`, `outK.epd`)
- Binary search + insertion sort used everywhere (O(n²))
- UTF-8 BOM stripping in every file

## Rust port

See `DESIGN.md` for the full design. Key decisions:

- **License**: MIT. No GPL dependencies. Write our own shallow parsers.
- **No external chess crates** — the PGN/EPD parsing 40H needs is surface-level (tag extraction, text tokenization), not semantic validation or move generation. ~900 lines of parsing total.
- **Board**: 8×8 array, not bitboards (tools inspect static positions, never search)
- **Architecture**: 5 crates — `40h-core`, `40h-pgn`, `40h-epd`, `40h-tools`, `40h-cli`
- **CLI**: Single `40h` binary with 92 flat subcommands, `clap` derive, `-i`/`-o` for I/O, stdin/stdout piping
- **Tool functions**: Pure functions on iterators, no file I/O. The CLI layer handles files.

## Related KV chess projects

- `~/projects/chess/chess-core/` — Rust library wrapping shakmaty (GPL-3). PGN parsing with variation trees. Used by ChessKit.
- `~/projects/chess/cow/` — Tauri desktop app. Uses shakmaty + pgn-reader. Has Polyglot hashing, SQLite position DB.
- `~/projects/chess/jja/` — Rust CLI chess toolkit. PGN filtering, ECO, opening books. Also uses shakmaty.
- `~/projects/chess/EPD-Analyzer/` — Go EPD→PGN converter with engine analysis.

---
> Source: [kayvee42/chxss](https://github.com/kayvee42/chxss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
