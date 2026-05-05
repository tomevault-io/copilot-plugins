## quran-sh

> A CLI tool for interacting with the Quran, built with Bun and TypeScript.

# quran.sh

A CLI tool for interacting with the Quran, built with Bun and TypeScript.

## Intent Layer

**Before modifying code in a subdirectory, read its AGENTS.md first** (if exists) to understand local patterns and invariants.

- **Source Code**: `src/` - Core logic and entry point.

### Global Invariants

- All source code must reside in the `src/` directory.
- Use `bun` for all tasks (running, installing, testing).
- Maintain strict TypeScript compliance.
- The project entry point is `src/index.ts`.

### Anti-patterns

- Avoid using `npm` or `yarn`; prefer `bun`.
- Do not add logic directly to the root directory.

---
> Source: [smashah/quran.sh](https://github.com/smashah/quran.sh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
