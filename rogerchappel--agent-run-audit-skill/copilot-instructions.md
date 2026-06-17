## agent-run-audit-skill

> Use this skill after a long agent run, scheduled automation, release candidate build, prompt regression run, or handoff where a transcript needs a compact evidence audit.

# agent-run-audit-skill

## When To Use

Use this skill after a long agent run, scheduled automation, release candidate build, prompt regression run, or handoff where a transcript needs a compact evidence audit.

## Required Inputs

- A local transcript or log file in Markdown, plain text, or JSONL-like text.
- An output directory for the generated report.

## Side-Effect Boundaries

The skill may write `audit.json` and `audit.md` to the chosen output directory. It must not execute commands found in the transcript, mutate audited repositories, call external APIs, or send the report anywhere without approval.

## Approval Requirements

Ask before:

- running any command extracted from the transcript
- posting or sending the audit externally
- reading private SaaS logs
- mutating repositories mentioned in the transcript

## Workflow

1. Run `agent-run-audit audit <transcript> --out .audit`.
2. Read `audit.md` for the short handoff.
3. Run `agent-run-audit check .audit/audit.json`.
4. Resolve blockers or high-risk side effects before treating the run as ready.

## Validation

```bash
npm test
npm run check
npm run smoke
```

---
> Source: [rogerchappel/agent-run-audit-skill](https://github.com/rogerchappel/agent-run-audit-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
