## granite

> For repository architecture, product boundaries, commands, and test workflow, see [CLAUDE.md](CLAUDE.md).

# AGENTS.md

For repository architecture, product boundaries, commands, and test workflow, see [CLAUDE.md](CLAUDE.md).

Codex-specific notes:

- No operational differences from the Claude workflow.
- This repo is TypeScript + Node ESM. Keep CLI wrappers thin and put business logic in `src/core/`.
- Preserve the product boundary: Granite is deterministic local markdown infrastructure, not an embedded LLM, vector store, scheduler, or autonomous agent.

---
> Source: [The-Vibe-Company/Granite](https://github.com/The-Vibe-Company/Granite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
