## demopilot

> Guidance for coding agents working in this repository.

# CLAUDE.md

Guidance for coding agents working in this repository.

## Demo-script format is the source of truth

The demo-script format lives in **`packages/core/src/schema.ts`** (a Zod schema,
with a JSON Schema export in `packages/core/src/jsonSchema.ts`). It is the single
source of truth for what a valid demo script looks like.

### If you change the script format, regenerate the skill

The `demopilot-demo-scripts` skill (`.claude/skills/demopilot-demo-scripts/`)
teaches agents how to author demo scripts. Its `references/script-schema.md` and
`templates/*.yaml` are **generated from the schema** — never hand-edit them.

Whenever you modify `packages/core/src/schema.ts` (or `jsonSchema.ts`), run:

```bash
pnpm generate:skill-docs
```

and commit the regenerated files alongside your schema change. In Claude Code
sessions this happens automatically via the `PostToolUse` hook in
`.claude/settings.json`; other agents and CI must run it manually so the skill
never drifts from the format it documents.

## Authoring demo scripts

When writing or editing demo scripts, use the `demopilot-demo-scripts` skill —
it covers accessibility-first targeting, action groups, `waitFor` for
reproducibility, narration, zoom/defaults tuning, and the MCP render workflow.

---
> Source: [MaverickEmanuel/DemoPilot](https://github.com/MaverickEmanuel/DemoPilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
