## qa-workflow

> A multi-agent QA workflow for Claude Code. Each QA task is handled by a specialized agent

# QA Workflow — Claude Code Instructions

@context/CONTEXT.md

## What this project is

A multi-agent QA workflow for Claude Code. Each QA task is handled by a specialized agent
in `.skills/`. Project context is auto-loaded from `context/CONTEXT.md` above.

## How to invoke agents

```bash
# Full pipeline — checkout PR, run app, validate in browser, review diff, generate tests
"Run the full QA pipeline for PR #2 on medusa-backend and PR #2 on medusa-storefront"

# Direct invocation
"Use the environment-manager skill to checkout feature/ISSUE-1-promotions-api and start the app"
"Use the functional-reviewer skill on this AC and diff: ..."
"Use the test-scenario-designer skill on these ACs: ..."
"Use the automation-writer skill. Read from qa-output/test-scenarios.md"
"Use the manual-validator skill. Read from qa-output/test-scenarios.md"
"Use the bug-reporter skill. Read from qa-output/functional-review.md"
"Validate the feature in browser using Chrome MCP"
```

## Agent map

| Skill | Model | Reads | Writes |
|---|---|---|---|
| Orchestrator | `claude-sonnet-4-6` | ticket + inputs | `qa-output/plan.md` |
| Environment Manager | `claude-sonnet-4-6` | PR/branch info | `qa-output/environment-status.md` |
| Functional Reviewer | `claude-opus-4-6` | AC + diff + browser findings | `qa-output/functional-review.md` |
| Bug Reporter | `claude-sonnet-4-6` | `functional-review.md` + `browser-validation.md` | `qa-output/bug-reports.md` |
| Test Scenario Designer | `claude-sonnet-4-6` | AC | `qa-output/test-scenarios.md` |
| Automation Writer | `claude-sonnet-4-6` | `test-scenarios.md` | `qa-output/automation/` |
| Manual Validator | `claude-sonnet-4-6` | `test-scenarios.md` | `qa-output/validation-report.md` |

## Execution flow

```
1. environment-manager
   └── Checkout branches, start app, health check

2. (parallel)
   ├── functional-reviewer ──→ qa-output/functional-review.md
   └── test-scenario-designer ──→ qa-output/test-scenarios.md

3. browser-validation (Chrome MCP)
   └── Navigate app, execute scenarios, verify ──→ qa-output/browser-validation.md

4. (conditional, parallel)
   ├── bug-reporter (if gaps found) ──→ qa-output/bug-reports.md
   └── automation-writer ──→ qa-output/automation/
```

## Output chaining

Every agent saves output to `qa-output/`. The next agent reads from that file.
Never paste raw output between agents — always reference the file path.

## Key principle: Live validation

The workflow is designed around **testing running code**, not just reading diffs.
The environment-manager checks out the PR branch and runs the application locally.
Browser validation happens against the actual running feature, not theoretical analysis.
This means findings are grounded in observable behavior, not just code review.

## Verification

Before running any agent, define what "done" looks like.
If output looks wrong, stop — do not continue the chain with bad input.
Use `/clear` between unrelated tickets to avoid context bleed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Anasss)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Anasss)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
