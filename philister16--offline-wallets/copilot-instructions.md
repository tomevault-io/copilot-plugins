## offline-wallets

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CLI utility (`ofwl`) for creating and restoring Ethereum wallets offline. Single-file Node.js app using ethers.js v6 and Node's native `crypto` module. Installed globally via npm and invoked as `ofwl <command>`.

## Commands

- `npm install` — install dependencies
- `node index.js create` — create a new wallet (or `ofwl create` if installed globally)
- `node index.js restore` — restore wallet from mnemonic (interactive prompts)
- `node index.js passphrase [bytes]` — generate random passphrase (default 16 bytes)

No build step, no tests, no linter configured.

## Architecture

Everything lives in `index.js` — a single script with a shebang line. It parses `process.argv` directly (no argument parsing library) and branches into commands: `create`, `restore`, `passphrase`, `--help`, `--version`. The `restore` command uses readline for interactive input; all others are non-interactive.

Key dependency: `ethers` (devDependency) provides `Mnemonic` and `HDNodeWallet` for BIP-39 mnemonic generation and HD wallet derivation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/philister16) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
