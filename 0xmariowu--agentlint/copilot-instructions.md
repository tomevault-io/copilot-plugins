## agentlint

> This fixture intentionally contains contradictory/vague/dead-weight

# ContradictionFixture

This fixture intentionally contains contradictory/vague/dead-weight
rules so downstream AI sub-agents (when tested against a real model)
have something to flag. The fixture-level test doesn't invoke an AI;
it only verifies that deep-analyzer produces well-formed D1/D2/D3
prompt tasks against this file.

## Contradictions

- Always use TypeScript for new code.
- Always use JavaScript for new code.
- Never commit directly to main.
- Commit directly to main for hotfixes.

## Dead weight

- Write correct code.
- Don't introduce bugs.
- Be careful when editing files.

## Vague rules

- Use good judgment.
- Do the right thing.
- Handle errors appropriately.

---
> Source: [0xmariowu/AgentLint](https://github.com/0xmariowu/AgentLint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
