## freedomtrader

> **Every coding session MUST update `DEVLOG.md` at the project root.**


# Freedom Trader — Agent Instructions

## Development Log (MANDATORY)

**Every coding session MUST update `DEVLOG.md` at the project root.**

- **Session start**: Read `DEVLOG.md` to understand recent context
- **Session end**: Append a session entry summarizing changes, decisions, and next steps
- If `DEVLOG.md` doesn't exist, create it
- Never overwrite existing entries — always append
- Use the `dev-docs` skill for entry format and templates

This is non-negotiable. No commit should happen without a corresponding DEVLOG entry.

## Code Change Standards

When making code changes in this project:

1. **Read before edit** — Always read the file and understand its context before modifying
2. **Preserve architecture** — Follow the existing module structure (see README.md for architecture)
3. **Chain awareness** — This is a dual-chain (BSC + Solana) project; changes may affect both chains
4. **Test in context** — Verify builds pass: `cd trader-extension && npm run build`
5. **Security first** — Never log or expose private keys, mnemonics, or sensitive wallet data

---
> Source: [dalangdalang934/freedomtrader](https://github.com/dalangdalang934/freedomtrader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
