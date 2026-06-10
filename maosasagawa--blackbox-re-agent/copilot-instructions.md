## blackbox-re-agent

> Cross-harness guide. This file is read by **pi**, **OpenAI Codex**, and (via

# AGENTS.md — revagent (black-box program logic analysis)

Cross-harness guide. This file is read by **pi**, **OpenAI Codex**, and (via
`CLAUDE.md`) **Claude Code**. It tells any coding agent how to use this repo's
reverse-engineering toolchain to analyze unknown binaries (apk, .o, .hex, exe, ELF,
PE, Mach-O, firmware). **Static analysis only — never execute the target sample.**

## Setup (once)
Run `./install.sh` — it provisions the toolchain (radare2, Ghidra headless, angr,
jadx, apktool, binwalk, capstone/lief/…), auto-installs pi if missing (`--no-pi` to
skip), and writes a capability manifest to `~/.pi/agent/revagent/tools.json`.

## How to analyze a file (works in ANY harness via bash)
The agent assets are deployed to `~/.pi/agent/revagent/` (`$H` below). Use the
helper CLIs + the system tools directly:

1. **Triage first** — identify format/arch/packing and get a plan:
   `python3 $H/bin/triage.py <file>`
2. **Unpack** when needed:
   - apk → `apktool d -f -o <out> <file>` and `jadx -d <out> <file>`
   - Intel-HEX → `python3 $H/bin/hex2bin.py <file.hex>` (reports load base)
   - firmware → `binwalk -e <file>` ; UPX → `upx -d -o <out> <file>`
3. **Surface map** — `python3 $H/bin/strings_scan.py <file>` ; `rabin2 -I -i -E -s <file>`
4. **Disassemble** — `radare2 -q -c 'aa; s sym.main; pdf' <file>` ; raw blobs →
   `python3 $H/bin/capstone_disasm.py <file> --arch arm --thumb --base 0x8000000`
5. **Decompile** (pseudo-C) — Ghidra headless:
   `$H/vendor/ghidra_*/support/analyzeHeadless /tmp/proj p -import <file> \
    -scriptPath $H/bin -postScript ghidra_decompile.py func <name|0xADDR> -deleteProject`
6. **Symbolic execution** (solve input-gated logic: license/password/seed-key):
   `python3 $H/bin/symexec_find.py <file> --find 0xSUCCESS --avoid 0xFAIL --stdin 32`

Set `$H=~/.pi/agent/revagent`. Where `tools.json` marks a backend missing, fall back
(rizin→objdump→capstone) or run `./install.sh` to provision it.

## Skills
`agent/skills/` contains [Agent Skills](https://agentskills.io)-standard workflows
(triage router, ELF/.o, PE/exe, Android apk, firmware/Intel-HEX). The installer
copies them into `~/.claude/skills/` and `~/.codex/skills/` when those harnesses are
present, so Claude Code / Codex load them natively. Read the matching `SKILL.md` for
the deep workflow.

## System prompt
`agent/system.md` is a ready-made RE-specialist system prompt — load it as your
system/append prompt for best results.

## pi-native usage
With pi: just run `revagent <file>` and chat — the 8 registered tools (triage,
unpack, strings_scan, symbols, disasm, decompile, symexec, provision) are called for
you. See `README.md`.

## Scope
Authorized analysis only (malware research, CTF, supplier/firmware security audits,
your own software). Reconstruct and explain logic; do not produce operational
malware, weaponized exploits, or detection-evasion tooling.

---
> Source: [maosasagawa/blackbox-re-agent](https://github.com/maosasagawa/blackbox-re-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
