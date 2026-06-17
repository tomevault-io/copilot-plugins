## agent-skill-index

> Use this skill when an agent needs to choose from, review, or document a local collection of reusable skills.

# agent-skill-index

Use this skill when an agent needs to choose from, review, or document a local collection of reusable skills.

## Required Tools

- Local filesystem read access to the skill root.
- Node.js 20 or newer for the CLI.

## Inputs

- A skill directory where each child folder may contain `SKILL.md`.
- Optional output paths for JSON and Markdown catalog files.

## Side-Effect Boundaries

- Read-only except for explicit `--out` and `--docs` targets.
- No network calls.
- No package installation or skill execution.

## Approval Requirements

- No approval is needed for local fixture scans.
- Ask for approval before indexing private skill directories that may contain sensitive workflow instructions.
- Ask for approval before committing generated catalogs into another repository.

## Examples

```bash
agent-skill-index ~/.codex/skills --out skill-index.json --docs SKILLS.md
agent-skill-index ./skills --fail-on-warnings
```

## Validation Workflow

1. Run the CLI against fixture skills.
2. Inspect warnings for missing safety or approval metadata.
3. Run `npm test`, `npm run check`, and `npm run smoke`.

---
> Source: [rogerchappel/agent-skill-index](https://github.com/rogerchappel/agent-skill-index) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
