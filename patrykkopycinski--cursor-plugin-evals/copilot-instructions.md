## yaml-conventions

> >-


# plugin-eval.yaml Conventions — Mandatory

These conventions are NON-NEGOTIABLE. The eval framework's Zod schema validates
BEFORE snake-to-camel conversion, so camelCase field names silently fail.

## 1. Field Names: ALWAYS snake_case

The Zod config schema uses snake_case. After validation, a `snakeToCamel` pass converts
them for the TypeScript runtime. If you write camelCase in YAML, Zod silently strips
the unknown keys and the field has no effect.

| WRONG (camelCase) | CORRECT (snake_case) |
|---|---|
| `pluginRoot` | `plugin_root` |
| `buildCommand` | `build_command` |
| `judgeModel` | `judge_model` |
| `expectedTools` | `expected_tools` |
| `requireEnv` | `require_env` |
| `expectError` | `expect_error` |
| `minimalEnv` | `minimal_env` |
| `responseContains` | `response_contains` |
| `responseNotContains` | `response_not_contains` |
| `toolArgs` | `tool_args` |
| `maxTurns` | `max_turns` |
| `firstTryPassRate` | `first_try_pass_rate` |
| `phaseGate` | `phase_gate` |

**Exception:** Keys inside `env:`, `args:`, and `assert.value` are pass-through — keep
them in whatever case the target API expects.

## 2. Assertion Dot Paths: Use Dot Notation for Array Indices

The integration assertion `resolveDotPath` splits on `.` and handles numeric segments
as array indices. Bracket notation `[N]` is also supported but dot notation is preferred.

| Preferred | Also valid |
|---|---|
| `content.0.text` | `content[0].text` |
| `content.0.type` | `content[0].type` |
| `hits.hits.0._source.name` | `hits.hits[0]._source.name` |

## 3. Scoring Weights: Must Be ≤ 1.0

The Zod schema validates `scoring.weights.*` with `z.number().max(1)`. Values > 1.0
cause a config validation error.

```yaml
# WRONG — will fail validation
scoring:
  weights:
    unit: 1.5
    integration: 2.0

# CORRECT
scoring:
  weights:
    static: 0.5
    unit: 0.8
    integration: 1.0
    llm: 1.0
    performance: 0.5
```

## 4. Environment Variables: No Bash-Style Defaults

The config interpolation does a simple `${VAR_NAME}` → `process.env[VAR_NAME]` lookup.
It does NOT support bash-style defaults like `${VAR:-default}`.

```yaml
# WRONG — throws "Unresolved environment variable: MY_URL:-http://localhost:9220"
env:
  MY_URL: "${MY_URL:-http://localhost:9220}"

# CORRECT — use plain env var references
env:
  MY_URL: "${MY_URL}"
  MY_API_KEY: "${MY_API_KEY}"
```

Set actual defaults in `.env.test` instead and load it before running evals.

## 5. The `adapter` Section Is NOT Used

The core runner ignores any top-level `adapter:` field. It constructs the MCP client
from `plugin.entry`, `plugin.transport`, and `plugin.env`. Do not add an `adapter:`
section — it has no effect and creates confusion.

## 6. Required Plugin Fields for Integration/Performance

For integration and performance layers to work, the plugin section MUST have:

```yaml
plugin:
  name: my-plugin
  dir: .
  entry: path/to/server.js     # REQUIRED for integration + performance
  build_command: npm run build  # snake_case!
  env:                          # Forwarded to the MCP server process
    MY_SERVICE_URL: "${MY_SERVICE_URL}"
```

## 7. Auto-Fix Checklist

When writing or reviewing ANY plugin-eval.yaml, automatically fix:

1. ✅ Convert ALL camelCase keys to snake_case
2. ✅ Convert ALL `content[0].text` paths to `content.0.text`
3. ✅ Ensure all scoring weights are ≤ 1.0
4. ✅ Remove any `${VAR:-default}` patterns — use plain `${VAR}`
5. ✅ Remove any `adapter:` section
6. ✅ Ensure `plugin.entry` exists (not just `plugin.build_command`)
7. ✅ Ensure `ci:` section exists with at least `score.avg` and `required_pass`

---
> Source: [patrykkopycinski/cursor-plugin-evals](https://github.com/patrykkopycinski/cursor-plugin-evals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
