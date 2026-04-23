## sagmal

> You are a programmer with expertise in modern TypeScript and modern Node.js.

# CLAUDE.md

## Context

You are a programmer with expertise in modern TypeScript and modern Node.js.
You are extremely competent with respect to enhancing and retaining the quality of the code.
You are working on a small CLI tool that translates any input text by using the DeepL API.

## Project Overview

- Overview and features of this tool: @README.md
- Development rules: @docs-internal/development.md
- Architecture overview: @docs-internal/architecture.md
- Documentation for npm libraries: see `docs-deps/` directory, e.g. `docs-deps/deepl-node/README.md` for DeepL Node.js library.

## Important Notes

- Target format is ESM and target platform is Node.js v22+, so prefer using modern Node.js features.
- The CLI command is designed at simplicity-first, so the format is not very common:
    - the language options are colon-separated instead of using `--from` and `--to` options.
    - the input text does not need to be quoted, so it can contain spaces.
- Npm dependencies are `deepl-node` and `dotenv`. Suggest adding more if necessary.

## Security Guidelines

- **Never log or expose API keys**: Avoid any console output, debug statements, or error messages that could expose the DeepL API key or other sensitive information. The API key should only be used for its intended purpose (DeepL API authentication) and never be visible in logs, error traces, or debugging output.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fal-works) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
