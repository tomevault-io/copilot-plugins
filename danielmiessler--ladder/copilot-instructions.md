## ladder

> Ladder is an open source pipeline for systematic improvement: Sources → Ideas → Hypotheses → Experiments → Results, with results feeding back as sources (the loop).

# Ladder — AI Guidance

## What Is This?

Ladder is an open source pipeline for systematic improvement: Sources → Ideas → Hypotheses → Experiments → Results, with results feeding back as sources (the loop).

## Key Conventions

- **IDs**: Each collection uses a prefix: `SR-`, `ID-`, `HY-`, `EX-`, `AL-`, `RE-` followed by 5-digit zero-padded number
- **Format**: All entries are markdown with YAML frontmatter
- **Status values**: `draft`, `active`, `testing`, `complete`, `archived`
- **Linking**: Entries reference upstream items (e.g., a hypothesis links to its parent idea via `idea: ID-00001`)
- **CLI**: Use `bun run ladder <command>` — never npm/npx

## Directory Structure

- `Sources/` — Raw inputs (papers, articles, observations, telemetry)
- `Ideas/` — Candidate solutions generated from sources
- `Hypotheses/` — Testable predictions from ideas
- `Experiments/` — Structured tests with methodology
- `Algorithms/` — Proven methods for specific tasks
- `Results/` — Verified outcomes from experiments
- `Tools/` — TypeScript CLI tooling (bun-based)

## When Adding Entries

1. Use the CLI: `bun run ladder add <type> --title "..."`
2. Or create manually following the TEMPLATE.md in each directory
3. Always include all required frontmatter fields
4. Link to upstream entries where applicable
5. Use the next available ID number

## Pipeline Flow

```
Sources → Ideation → Ideas → Hypotheses → Experiments → Results
   ↑                                                        |
   └──────────────────────── Loop ──────────────────────────┘
```

Results feed back as sources. This is the core mechanic.

## Code Style

- TypeScript only, bun runtime
- No Python, no npm/npx
- Minimal dependencies
- CLI-first philosophy

---
> Source: [danielmiessler/Ladder](https://github.com/danielmiessler/Ladder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
