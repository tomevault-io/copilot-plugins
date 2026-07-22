## pi-llm-wiki

> Instructions for AI agents working on this codebase.

# AGENTS.md

Instructions for AI agents working on this codebase.

## Project

`@zosmaai/pi-llm-wiki` — A pi package that implements Andrej Karpathy's LLM Wiki pattern as a self-maintaining knowledge base.

## Tech Stack

- TypeScript (ES2022, ESM)
- Vitest for testing
- Biome for linting/formatting
- GitHub Actions for CI
- npm for publishing

## File Layout

```
├── extensions/llm-wiki/     # TypeScript extension (12 tools + guardrails)
│   ├── index.ts             # Entry point
│   └── lib/                 # tools.ts, metadata.ts, guardrails.ts, utils.ts, source-packet.ts
├── skills/llm-wiki/         # SKILL.md + templates
├── prompts/                 # 8 slash command templates
├── test/                    # Vitest tests
├── docs/                    # Documentation
└── scripts/                 # release.js
```

## Conventions

- Use `node:fs/promises` for async file I/O, not sync
- Prefer small, pure functions in `lib/`
- Extension tools must have: name, label, description, promptSnippet, promptGuidelines, parameters (TypeBox), execute
- Guardrails block `.llm-wiki/raw/**` and `.llm-wiki/meta/**` edits at the tool_call hook level
- Metadata auto-rebuilds on `turn_end` after `.llm-wiki/wiki/**` edits
- Source IDs: `SRC-YYYY-MM-DD-NNN`
- Page filenames: `kebab-case.md`
- Wikilinks: folder-qualified, e.g. `[[concepts/retrieval-augmented-generation]]`

## Testing

```bash
pnpm test              # run tests
pnpm test:coverage     # run with coverage
pnpm typecheck         # TypeScript check
pnpm lint              # biome check
```

## Release

```bash
npm run release:patch  # or minor/major
npm run release:push   # push tags
```

Never edit `package.json` version manually — use the release script.

---
> Source: [zosmaai/pi-llm-wiki](https://github.com/zosmaai/pi-llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
