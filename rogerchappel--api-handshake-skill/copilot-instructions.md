## api-handshake-skill

> Use this skill before implementing an API connector or webhook workflow when the agent needs a precise handshake plan and test fixtures.

# api-handshake-skill

Use this skill before implementing an API connector or webhook workflow when the agent needs a precise handshake plan and test fixtures.

## Required Tools

- Local filesystem access to the spec or notes directory.
- Node.js 20 or newer.

## Inputs

- OpenAPI snippets, Markdown notes, schema examples, webhook docs, or plain text requirements.
- Optional output path for the generated plan or fixtures.

## Side-Effect Boundaries

- Local-first and offline by default.
- Reads only the input path provided by the user.
- Writes only explicit output files or directories.
- Does not call APIs, validate credentials, or register webhooks.

## Approval Requirements

- Ask before using real credentials or production examples.
- Ask before making any live API request.
- Ask before sending generated fixtures to an external service.

## Examples

```bash
api-handshake-skill plan specs/ --out integration-plan.md
api-handshake-skill fixtures integration-plan.md --out fixtures/
```

## Validation Workflow

1. Run the plan command against local fixtures.
2. Confirm all missing sections are either fixed or accepted as known gaps.
3. Generate fixtures and review placeholders before using them in tests.
4. Run `npm test`, `npm run check`, and `npm run smoke`.

---
> Source: [rogerchappel/api-handshake-skill](https://github.com/rogerchappel/api-handshake-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
