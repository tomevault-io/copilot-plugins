## pi-evo-research

> Population-guided evolutionary research for coding agents. pi extension + skills.

# pi-evo-research — Agent Notes

Population-guided evolutionary research for coding agents. pi extension + skills.

## Project type
pi-extension package. No build step. TypeScript loaded directly by pi.

## Running tests
```bash
npm test
```
Uses node --experimental-strip-types --test. Requires Node >=22.

## Key files
- `extensions/pi-evo-research/index.ts` — Main extension (~3150 lines): tools, dashboard, auto-resume, compaction
- `extensions/pi-evo-research/population.ts` — Population state: scheduler, candidate management
- `extensions/pi-evo-research/jsonl.ts` — JSONL log parser/reconstructor
- `extensions/pi-evo-research/hooks.ts` — Hook runner for before.sh/after.sh
- `extensions/pi-evo-research/compaction.ts` — Deterministic compaction summary builder
- `extensions/pi-evo-research/shortcuts.ts` — Keyboard shortcut config resolver
- `skills/pi-evo-research-create/SKILL.md` — Setup skill for session creation
- `skills/pi-evo-research-finalize/SKILL.md` — Finalize skill for branch cleanup
- `skills/pi-evo-research-hooks/SKILL.md` — Hook authoring skill with examples
- `assets/template.html` — Self-contained live dashboard HTML

## Dependencies
- Peer deps only: @earendil-works/pi-coding-agent, pi-ai, pi-tui, typebox
- Dev deps match peers
- No production deps

## Structured docs
Canonical YAML artifacts at the resolved structured docs root. See repo/ for architecture, data model, invariants, dependency rules, risks, and agent operating guide.

## Publishing
1. Bump version in package.json
2. Add CHANGELOG section
3. Run npm test
4. Push to main
5. Publish manually: npm publish --access public

## Constraints
- No production dependencies
- No build steps
- No editing jsonl or population files manually
- METRIC name=value format is fixed contract
- JSONL is append-only; population is atomically written

---
> Source: [PriNova/pi-evo-research](https://github.com/PriNova/pi-evo-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
