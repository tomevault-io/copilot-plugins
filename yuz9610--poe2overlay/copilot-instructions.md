## encoding

> Text encoding, BOM, and line endings for all project files


# Encoding conventions

Read `docs/ENCODING.md` for the full policy and tooling matrix.

When creating or editing files in this repo:

- Use **UTF-8 without BOM** for `.rs`, `.ts`, `.tsx`, `.js`, `.mjs`, `.json`, `.html`, `.css`, `.md`, `.toml`, `.ps1`, and everything under `data/`.
- Run PowerShell scripts only with **PowerShell 7+** (`pwsh`); do not target Windows PowerShell 5.1. New `.ps1` files should include `#Requires -Version 7.0` and dot-source `scripts/_InitializeUtf8Console.ps1` after the `param` block.
- Do **not** save Chinese (or any) text as GBK, Big5, or system ANSI.
- Do **not** enable encoding auto-guess; preserve existing UTF-8 bytes in `data/*.json` so area names match `Client.txt` exactly.
- Line endings: **LF** for all files including `.ps1`; only `.bat` / `.cmd` use CRLF (per `.editorconfig`).
- Node scripts: read/write with `"utf8"`; strip a leading BOM with `.replace(/^\uFEFF/, "")` when reading JSON if needed.
- Rust already assumes UTF-8 for `read_to_string` and `from_utf8_lossy` on logs; do not introduce other encodings without explicit project approval.

If a file shows mojibake or `` replacement characters, stop and fix encoding before editing content.

---
> Source: [yuz9610/POE2Overlay](https://github.com/yuz9610/POE2Overlay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
