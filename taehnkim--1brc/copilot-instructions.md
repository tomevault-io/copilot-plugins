## 1brc

> This is a single-purpose **Bun + TypeScript CLI** for the 1 Billion Row Challenge (1BRC). It is a one-shot batch program — there are **no servers, services, ports, or databases** to run. You invoke it against an input file and it prints aggregated min/mean/max per weather station, then exits.

# AGENTS.md

## Cursor Cloud specific instructions

This is a single-purpose **Bun + TypeScript CLI** for the 1 Billion Row Challenge (1BRC). It is a one-shot batch program — there are **no servers, services, ports, or databases** to run. You invoke it against an input file and it prints aggregated min/mean/max per weather station, then exits.

### Runtime / environment
- `bun` is **not** preinstalled and `https://bun.sh/install` is network-blocked in this environment. It is installed via npm to `~/.local` (`npm install -g bun --prefix "$HOME/.local"`), which the startup update script keeps current. `~/.local/bin` is on `PATH` via `~/.bashrc`; if a non-interactive shell can't find `bun`, call it as `~/.local/bin/bun`.
- `.cursorrules` says to prefer Bun commands. Node also works as an alternate runtime.

### Common commands (see `package.json` scripts and `README.md`)
- Run (Bun, preferred): `bun run main.ts test.txt`
- Run (Node alternate): `npm run node test.txt`
- Typecheck: `bunx tsc --noEmit`
- Build: `bun run build` (emits to `dist/`, which is gitignored)
- There is **no lint or automated test framework** configured.

### Gotchas
- `test.txt` (~1000 rows) is the quick end-to-end input. The real `measurements.txt` (~13.8 GB) is **not** in the repo; generating it needs JDK 21 + the upstream `gunnarmorling/1brc` repo (see `README.md`) and is not required for normal development.
- `correct_answer.txt` is the expected output for the **full** `measurements.txt` and uses a different format (`station=min/mean/max`). Do **not** diff it against `test.txt` runs — the program's own output format is `station:min/mean/max`.

---
> Source: [taehnkim/1brc](https://github.com/taehnkim/1brc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
