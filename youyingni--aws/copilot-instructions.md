## aws

> This repo stores Traditional Chinese strings in TypeScript sources and runs on a

# Agent instructions

## File encoding on Windows (read this before editing any source file)

This repo stores Traditional Chinese strings in TypeScript sources and runs on a
machine whose ANSI code page is CP950 (zh-TW).

**Never round-trip a source file through PowerShell text cmdlets.** On 2026-08-01 this
command destroyed 4,746 lines of `dashboard-preview/src/MapPanel.tsx`:

```powershell
(Get-Content -Path $path).ForEach({ [void]$lines.Add($_) })   # no -Encoding -> CP950
$lines[3734] = '...'
Set-Content -Path $path -Value $newLines -Encoding UTF8
```

PowerShell 5.1 defaults `Get-Content`, `Set-Content`, `Out-File` and `Add-Content` to
the ANSI code page, not UTF-8. The read above decoded UTF-8 as CP950, turning every
Chinese character into Big5 mojibake and eating the byte after each failure — usually a
closing quote. 1,051 compile errors, unrecoverable in place. Forensics in `_recover/`.

Rules:

- Edit source files with Node (`fs.readFileSync(p, 'utf8')`) or the editor, not shell
  text pipelines.
- If a shell text cmdlet is unavoidable, put `-Encoding utf8` on **every** read and
  write. One omission loses the file.
- Avoid index-based line splicing (`$lines[3734] = ...`, `GetRange`/`InsertRange`); it
  orphans JSX closing tags when the replaced range and replacement differ in length.
  Use targeted string replacement on unique context instead.
- Do not trust console output to judge encoding. The terminal also misrenders UTF-8 as
  CP950, so correct data can look corrupt and corrupt data can look correct. Verify by
  reading the file with Node as UTF-8.

## Verify before you finish

```bash
node scripts/check-encoding.mjs                                  # exit 2 = mojibake
cd dashboard-preview && npx tsc --noEmit -p tsconfig.app.json    # must be 0 errors
cd dashboard-preview && npm run build
```

## Dev server

```bash
cd dashboard-preview && npm run dev     # http://localhost:5173/
```

Requires `VITE_MAPBOX_TOKEN` in `dashboard-preview/.env.local`.

## macOS

- On macOS, if `gh` returns HTTP 401 in the normal sandbox, verify authentication from an approved unsandboxed execution context before asking the user to log in again; the sandbox may be unable to read credentials stored in Keychain.

---
> Source: [youyingni/AWS](https://github.com/youyingni/AWS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
