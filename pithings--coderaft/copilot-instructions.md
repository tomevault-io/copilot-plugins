## coderaft

> Custom VS Code server build with bundled dependencies.

# coderaft

Custom VS Code server build with bundled dependencies.

## Project Structure

- `lib/` — Main dependency package, pulls in `code-server@^4.114.0` and all required VS Code native/runtime deps
- `shims/` — Workspace shim packages replacing native deps (pnpm overrides). See [shims/README.md](./shims/README.md) for the full list, pnpm override mappings, and native binary breakdown.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pithings)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pithings)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
