## openclaw-defender

> - **Do NOT maintain backward compatibility** unless explicitly requested. Break things boldly.

# Agent Guidelines

## Core Principles

- **Do NOT maintain backward compatibility** unless explicitly requested. Break things boldly.
- Zero runtime dependencies. Node.js 18+ built-ins only.
- All imports use `.js` extension (ESM). Use `import type` for type-only imports.

---

## Project Overview

**Project type:** npm library (3-layer prompt injection defence for chat bots)
**Primary language:** TypeScript (ESM, NodeNext)
**Key dependencies:** None (discord.js is optional peerDep). vitest for testing.

---

## Commands

```bash
npm run build    # tsc -p tsconfig.json
npm test         # vitest run
npm run test:watch  # vitest
```

---

## Code Conventions

- Follow existing patterns in the codebase
- Prefer explicit over clever
- Delete dead code immediately

---

## Architecture

```
src/
  rules/       # Layer 1: regex/keyword rules (0ms, 16 built-in rules)
  classifier/  # Layer 2: ML classifier adapters (Prompt Guard 2, DeBERTa, API)
  llm/         # Layer 3: LLM judgment + intent alignment (Cerebras default)
  actions/     # block / sanitize / wrap
  integrations/ # discordjs, openclaw, generic
  scanner.ts   # 3-layer pipeline orchestrator
  normalizer.ts # Unicode normalization (zero-width, fullwidth, NFKC)
```

---
> Source: [nyosegawa/openclaw-defender](https://github.com/nyosegawa/openclaw-defender) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
