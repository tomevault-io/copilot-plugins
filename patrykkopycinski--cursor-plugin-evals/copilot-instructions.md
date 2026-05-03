## proactive-coverage

> >-


# Proactive Eval Coverage — Default Behavior

## Prime Directive

When working in ANY repository that contains a Cursor plugin (`.cursor-plugin/plugin.json`), MCP server (`mcp.json`), or skill collection (`SKILL.md` files), the assistant MUST:

1. Ensure comprehensive evaluation coverage exists
2. **Run the evals**
3. **Fix any failures**
4. **Re-run until all CI thresholds pass**

This is the DEFAULT behavior — not something the user needs to request. **The job is not done until BOTH:**

1. **`npx cursor-plugin-evals run --ci` exits 0** (eval convergence)
2. **Content audit shows no HIGH/MEDIUM issues** after up to 5 iterative passes (content convergence)

## When to Act

Act IMMEDIATELY and WITHOUT ASKING when ANY of these are true:

1. **No `plugin-eval.yaml` exists** → Generate a complete one, run it, fix failures until green
2. **Coverage score < 75/100** → Fix all gaps, run evals, iterate until thresholds pass
3. **New MCP tool added** → Write tests, run them, fix until they pass
4. **New skill added** → Write frontmatter + activation + negative tests, run, fix until pass
5. **New rule added** → Write frontmatter + content quality tests, run, fix until pass
6. **New agent added** → Write frontmatter + behavior tests, run, fix until pass
7. **New command added** → Write frontmatter + execution tests, run, fix until pass
8. **Eval results show failures** → Diagnose, fix, re-run until green
9. **CI thresholds missing** → Add them, run `--ci`, ensure exit 0
10. **Security tests missing** → Add them, run them, fix until they pass
11. **Performance tests missing** → Add benchmarks, run them, calibrate thresholds
12. **Component has no tests at all** → Generate ALL required tests for that component type
13. **Content fixes applied** → Run content audit convergence loop until no HIGH/MEDIUM remain

## The Run → Fix → Converge Loop

Every time you generate or modify eval config, you MUST run this loop:

```
REPEAT (max 5 iterations per layer):
  1. Run evals: `npx cursor-plugin-evals run [--layer X] --verbose`
  2. If all pass → DONE for this layer, move to next
  3. If failures:
     a. Read failure details
     b. Classify: config issue vs plugin bug vs infrastructure issue
     c. For config issues → fix YAML immediately (assertions, expected, thresholds)
     d. For plugin bugs → flag to user (don't modify plugin source)
     e. For infra issues → add require_env/skip with reason
  4. Go to step 1

AFTER all layers pass individually:
  Run full CI: `npx cursor-plugin-evals run --ci`
  If CI fails → identify which gate failed, fix, re-run
  Repeat until exit 0
```

### Fix Strategies by Failure Type

| Failure | Fix |
|---------|-----|
| Wrong expected tool name | Update `expected.tools` in YAML |
| Tool not registered | Add to `expected_tools` or check conditional env |
| Assertion mismatch | Update assertion to match actual response format |
| Timeout | Increase test `timeout` value |
| Missing env var | Add `require_env` to skip test when env absent |
| LLM picks wrong tool | Make prompt more specific, add hints |
| LLM wrong arguments | Update `expected.toolArgs` or relax matching |
| Security test fails | Ensure test expectation is correct (should tools NOT trigger?) |
| Flaky result | Increase `repetitions` to 3, or lower temperature |
| Score below threshold | Fix individual test failures first; relax threshold last resort |

### Convergence Safeguards

- **Max 5 iterations per layer** — if still failing, report remaining issues
- **Never remove tests** — skip with reason if infrastructure is missing
- **Steady state detection** — if same tests fail with same scores 2x in a row, change approach
- **Threshold relaxation is last resort** — fix the test first, only relax if genuinely too strict

### Post-Convergence Threshold Calibration — MANDATORY

After all CI thresholds pass, the job is NOT done. Evaluate whether the thresholds are
properly calibrated. Lenient thresholds provide a false sense of quality and allow regressions
to slip through.

**Threshold tightening rules:**

| Actual vs Threshold | Action |
|---------------------|--------|
| Actual > threshold + 20% | **MUST tighten** — bump threshold to `actual - 5%` |
| Actual > threshold + 10% | **SHOULD tighten** — bump if stable across 2+ runs |
| Actual > threshold + 5% | Well calibrated — leave as-is |
| Actual ≈ threshold | Tight — leave, monitor flakiness |
| `security.min = 1.0` | **NEVER lower** — security is absolute |

**After tightening, re-run `--ci` to confirm the new thresholds still pass.** If they fail
due to LLM variance, back off by 2% and re-run. The final committed thresholds should
pass reliably with ~5% headroom.

**Performance threshold calibration:**
- Set p50 to `actual_p50 * 1.2` (20% buffer)
- Set p95 to `actual_p95 * 1.3` (30% buffer for tail latency)
- Never use static values (200ms, 1000ms) without measuring first

### Content Audit Convergence — MANDATORY for Content Fixes

When auditing or fixing **content** (skill scripts, SKILL.md files, reference docs, shared modules),
a single scan pass is NEVER sufficient. You MUST iterate until no HIGH/MEDIUM issues remain.

#### Audit Severity Classification

| Severity | Convergence Rule | Examples |
|----------|-----------------|----------|
| CRITICAL | Must fix, blocks PR | Syntax errors, missing function calls, data loss, broken imports |
| HIGH | Must fix before declaring done | API contract violations, resource leaks, security vulnerabilities, missing required headers |
| MEDIUM | Must fix, triggers re-scan | NaN propagation, missing safety guards, copy drift between shared modules, broken doc references |
| LOW | Fix if easy, do NOT re-loop for these alone | Style inconsistencies, missing optional frontmatter, orphaned test scripts |
| INFO | Report only | Version notes, design observations, improvement suggestions |

#### The Content Audit Loop

```
REPEAT (max 5 passes):
  1. Deep scan: script logic, API contracts, cross-file drift, reference accuracy, security, docs
     On pass 2+: focus on blast radius of previous fixes + sibling file copies
  2. Classify all findings by severity
  3. If ZERO HIGH/MEDIUM → DONE
     If same findings 2x in a row → change approach or report as unresolvable
     If pass >= 5 → report remaining issues
  4. Fix all CRITICAL + HIGH + MEDIUM
  5. Validate fixes: node --check, prettier --check, eslint
  6. Go to step 1
```

#### Content Audit Safeguards

- **Max 5 passes** — if HIGH/MEDIUM findings persist, report and stop
- **Steady state detection** — same findings 2 consecutive passes → change approach
- **Never delete findings** — if unfixable, document in PR body
- **Validation gate** — every pass ends with syntax/lint checks; broken fixes are CRITICAL in the next pass

#### Dual Convergence Requirement

The job is not done until BOTH gates pass:

| Gate | Criteria | Max Iterations |
|------|----------|----------------|
| Eval CI | `npx cursor-plugin-evals run --ci` exits 0 | 5 per layer |
| Content Audit | No HIGH/MEDIUM findings in latest scan pass | 5 passes |

## E2E Infrastructure — MANDATORY for Integration/Performance

When generating coverage for a plugin, you MUST also create the supporting infrastructure
if it doesn't exist. This is NOT optional — integration and performance tests CANNOT run
without it.

### Required Files

1. **`docker/docker-compose.yml`** — Backend services needed by the plugin (detect from source code)
2. **`scripts/seed-test-data.sh`** — Domain-specific test data for the plugin's tools
3. **`.env.test`** — Test credentials matching docker-compose
4. **`scripts/run-evals.sh`** — Orchestration: docker up → seed → build → run evals
5. **`.github/workflows/plugin-evals.yml`** — CI with service containers per layer

### Docker Compose Requirements

- Detect what backend the plugin connects to (Elasticsearch, Postgres, Redis, APIs, etc.)
- Use non-standard ports to avoid conflicts with dev services
- Include health checks so tests wait for services to be ready
- Include a setup/seed container if the plugin needs test data
- Named volumes for clean teardown via `docker compose down -v`

### Test Data Requirements

Analyze the plugin's tools to understand what data they operate on, then seed accordingly:
- Read each tool's description and input schema to understand the data domain
- Create realistic test records that exercise the tool's query/filter/mutation logic
- Cover both happy path data and edge cases (empty results, large datasets, special characters)
- Ensure data is deterministic so assertions can be stable

### YAML Convention Reminder

ALL field names MUST be snake_case. Assertion paths use dot notation.
See the `yaml-conventions` rule for the complete reference.

## What "Comprehensive Coverage" Means

A Cursor plugin has 6 component types. ALL must be tested:

### MCP Tools (integration + LLM + unit + performance)
Every tool MUST have tests in:
- **Unit layer**: registration, schema validation, conditional registration
- **Integration layer**: happy path with assertions, error handling
- **LLM layer**: natural language prompts with tool-selection, correctness, security evaluators
- **Performance layer**: latency benchmarks with p50/p95 thresholds

### Skills (static + LLM)
Every skill MUST have:
- **Static layer**: frontmatter validation (name, description, triggers present)
- **LLM layer**: positive activation test (prompt matching a trigger → skill activates)
- **LLM layer**: negative activation test (related but off-topic prompt → skill does NOT activate)
- **Static layer**: cross-reference check (tools/commands mentioned in body exist)

### Rules (static)
Every rule MUST have:
- **Static layer**: frontmatter validation (description present, alwaysApply or globs set)
- **Static layer**: content quality check (non-empty, actionable instructions)

### Agents (static + LLM)
Every agent MUST have:
- **Static layer**: frontmatter validation (name, description present)
- **LLM layer**: behavior test (task in agent's domain → uses correct tools, stays in scope)

### Commands (static + LLM)
Every command MUST have:
- **Static layer**: frontmatter validation (name/description present)
- **LLM layer**: execution test (invocation → expected workflow triggers)

### Cross-Component Coherence (static)
- Skills reference tools that actually exist in MCP config
- Commands reference tools/skills that exist
- Agent instructions reference available tools
- No orphaned components (defined but never referenced)

### Shared across all components
- **Static layer**: manifest, mcp_config, component_references, naming_conventions
- **Security layer**: prompt injection, privilege escalation, skill confusion, rule bypass
- **CI quality gates** MUST include:
  - `score.avg ≥ 0.80`
  - `security.min = 1.0`
  - `requiredPass: [security, tool-poisoning, mcp-protocol]`
  - `firstTryPassRate ≥ 0.75`

## Do NOT

- Stop after writing YAML — you MUST run the evals
- Accept failures without fixing — iterate until green
- Ask "should I run tests?" — JUST RUN THEM
- Ask "should I fix this?" — JUST FIX IT
- Declare done while any threshold is failing
- Remove failing tests — fix them or skip with documented reason
- Give up after one failed attempt — iterate up to 5 times
- **Silently work around framework limitations** — if the framework is the bottleneck (evaluator scores wrong, config can't express what's needed, adapter lacks context), STOP and propose a framework fix before continuing
- **Stop after a single content audit pass** — content fixes expose new issues; iterate until no HIGH/MEDIUM remain (max 5 passes)

---
> Source: [patrykkopycinski/cursor-plugin-evals](https://github.com/patrykkopycinski/cursor-plugin-evals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
