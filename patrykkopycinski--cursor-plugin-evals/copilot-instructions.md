## config-validation

> >-


# Config Validation — Auto-Fix on Save

When evaluation configuration files are modified, validate AND auto-fix issues.

## What to Check and Auto-Fix

1. **YAML syntax errors**: Report the exact line and error. Cannot auto-fix.

2. **Missing required fields**:
   - Test without `name` → generate a descriptive name from the test content
   - Integration test without `tool`/`args` → flag as error (cannot guess)
   - LLM test without `evaluators` → add default: `[tool-selection, correctness, mcp-protocol, security]`
   - LLM test without `expected` → add `expected: { tools: [] }` as placeholder

3. **Invalid evaluator names** (common typos):
   - `tool_selection` → auto-fix to `tool-selection`
   - `tool_args` → auto-fix to `tool-args`
   - `tool_sequence` → auto-fix to `tool-sequence`
   - `response_quality` → auto-fix to `response-quality`
   - Any evaluator not in the 24 known evaluators → flag as error

4. **Invalid layer names**: Must be one of: `unit`, `static`, `integration`, `llm`, `performance`, `skill`

5. **Missing CI thresholds**: If `ci:` section is absent, add default:
   ```yaml
   ci:
     score:
       avg: 0.80
     evaluators:
       security:
         min: 1.0
     requiredPass: [security, mcp-protocol]
     firstTryPassRate: 0.75
   ```

6. **LLM tests without security evaluator**: Append `security` to the evaluators list

7. **Unreasonable thresholds**: Warn if timeout > 300000ms or score thresholds > 1.0

8. **camelCase field names** (CRITICAL — silently ignored by Zod):
   - `expectedTools` → auto-fix to `expected_tools`
   - `requireEnv` → auto-fix to `require_env`
   - `expectError` → auto-fix to `expect_error`
   - `minimalEnv` → auto-fix to `minimal_env`
   - `buildCommand` → auto-fix to `build_command`
   - `pluginRoot` → auto-fix to `plugin_root`
   - `judgeModel` → auto-fix to `judge_model`
   - `toolArgs` → auto-fix to `tool_args`
   - `maxTurns` → auto-fix to `max_turns`
   - `responseContains` → auto-fix to `response_contains`
   - `responseNotContains` → auto-fix to `response_not_contains`
   - `firstTryPassRate` → auto-fix to `first_try_pass_rate`
   - `phaseGate` → auto-fix to `phase_gate`

9. **Bracket notation in assertion paths**: `content[0].text` → auto-fix to `content.0.text`

10. **Scoring weights > 1.0**: All `scoring.weights.*` values must be ≤ 1.0. Auto-fix by dividing by the max weight to normalize.

11. **Bash-style env defaults**: `${VAR:-default}` → auto-fix to `${VAR}`. Defaults belong in `.env.test`.

12. **Orphan `adapter:` section**: The runner ignores `adapter:` — remove it and ensure `plugin.entry` is set instead.

13. **Missing e2e infrastructure**: If `require_env` references service URLs (e.g., `ES_URL`, `DATABASE_URL`, `REDIS_URL`) but no `docker/docker-compose.yml` exists, warn that integration tests need infrastructure.

## When to Stay Silent

- Minor whitespace or comment changes
- Changes to non-eval YAML files
- If the file is already valid and complete

---
> Source: [patrykkopycinski/cursor-plugin-evals](https://github.com/patrykkopycinski/cursor-plugin-evals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
