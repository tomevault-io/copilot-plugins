## eval-suite-guidelines

> Guidelines for writing effective plugin-eval.yaml test suites


# Writing Plugin Eval Suites

## Suite Design Principles

1. **One concern per test** — test a single tool, assertion, or behavior
2. **Descriptive names** — names should explain what's being tested without reading config
3. **Layer-appropriate** — use the right layer for each concern:
   - Unit: schema validation, registration checks (no infrastructure needed)
   - Integration: tool execution, assertions, workflow chains (needs cluster)
   - LLM: agent quality, tool selection, response quality (needs LLM API)

## Integration Test Best Practices

- Always include at least one assertion per test
- Use `expect_error: true` for negative tests (invalid args, auth failures)
- Workflow chains should test realistic multi-step operations
- Use `$prev.field` for output variable binding between workflow steps

## LLM Eval Test Best Practices

- Write prompts as natural language questions (how a user would ask)
- Don't write prompts that hint at tool names
- Include `tool-selection` and `security` evaluators on every LLM test
- Set `max_turns: 5` unless the scenario requires more
- Start with `repetitions: 3` for statistical confidence

## Assertion Operators Reference

| Operator | Description | Example |
|---|---|---|
| eq | Exact equality | `{ field: "isError", op: eq, value: false }` |
| contains | String contains | `{ field: "content.0.text", op: contains, value: "cluster_name" }` |
| exists | Field is defined | `{ field: "content.0.text", op: exists }` |
| type | Check JS type | `{ field: "content", op: type, value: "array" }` |
| length_gte | Array length >= | `{ field: "content", op: length_gte, value: 1 }` |
| matches | Regex match | `{ field: "content.0.text", op: matches, value: "status.*green" }` |

## Evaluator Thresholds

Default recommendations:
- `tool-selection: 0.9` — strict: the right tool must be selected
- `tool-args: 0.7` — moderate: some args may be inferred differently
- `tool-sequence: 0.8` — strict-ish: order matters for workflows
- `response-quality: 0.7` — moderate: response content varies
- `mcp-protocol: 1.0` — absolute: protocol must always be correct
- `security: 1.0` — absolute: no credential leaks ever

---
> Source: [patrykkopycinski/cursor-plugin-evals](https://github.com/patrykkopycinski/cursor-plugin-evals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
