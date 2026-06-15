## local-model-autotuning

> <!-- Scope: repo development agents. Research loop agents → read program.md -->

# AutoResearch Agent Work Contract

<!-- Scope: repo development agents. Research loop agents → read program.md -->

## Purpose
This repository implements an autonomous hill-climbing search system optimizing localized LLM runtime flags.

## Ownership
Repository-wide agent guidelines are owned by the repository developers.

## Local Contracts
- Respond terse like smart caveman. All technical substance stay. Only fluff die.
- Loop agents: Strictly forbidden from editing code. If error/crash occurs, stop immediately, report error, warn user.
- Results local-only: Never push results, tweaks, or run branches to remote repository. Keep all benchmark runs offline.
- Architecture: Never overengineer. Keep it simple. Less is more. Reduce lines of code. Simplify instead of complicate.

## Work Guidance
- Use `/caveman lite|full|ultra|wenyan` for communication style constraint.
- Prioritize test-driven sanity. Verify logic changes using the test suite.
- Maintain config single source of truth in [config.py](file:///home/shark/workspace/autoresearch-public/autoresearch/core/config.py).

## Verification
- Test with `pytest`. Ensure all 101 tests pass.
- Inspect `results.tsv` to ensure it is not polluted or modified by agent logic.

## Child DOX Index
- [ClawBench/AGENTS.md](file:///home/shark/workspace/autoresearch-public/ClawBench/AGENTS.md) — Agency benchmark harness and sandboxing boundaries.

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **local-model-autoresearch** (630 symbols, 1034 relationships, 46 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> Index stale? Run `node .gitnexus/run.cjs analyze` from the project root — it auto-selects an available runner. No `.gitnexus/run.cjs` yet? `npx gitnexus analyze` (npm 11 crash → `npm i -g gitnexus`; #1939).

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows. For regression review, compare against the default branch: `detect_changes({scope: "compare", base_ref: "main"})`.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `context({name: "symbolName"})`.

## Never Do

- NEVER edit a function, class, or method without first running `impact` on it.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis.
- NEVER rename symbols with find-and-replace — use `rename` which understands the call graph.
- NEVER commit changes without running `detect_changes()` to check affected scope.

## Resources

| Resource | Use for |
|----------|---------|
| `gitnexus://repo/local-model-autoresearch/context` | Codebase overview, check index freshness |
| `gitnexus://repo/local-model-autoresearch/clusters` | All functional areas |
| `gitnexus://repo/local-model-autoresearch/processes` | All execution flows |
| `gitnexus://repo/local-model-autoresearch/process/{name}` | Step-by-step execution trace |

## CLI

| Task | Read this skill file |
|------|---------------------|
| Understand architecture / "How does X work?" | `.claude/skills/gitnexus/gitnexus-exploring/SKILL.md` |
| Blast radius / "What breaks if I change X?" | `.claude/skills/gitnexus/gitnexus-impact-analysis/SKILL.md` |
| Trace bugs / "Why is X failing?" | `.claude/skills/gitnexus/gitnexus-debugging/SKILL.md` |
| Rename / extract / split / refactor | `.claude/skills/gitnexus/gitnexus-refactoring/SKILL.md` |
| Tools, resources, schema reference | `.claude/skills/gitnexus/gitnexus-guide/SKILL.md` |
| Index, status, clean, wiki CLI commands | `.claude/skills/gitnexus/gitnexus-cli/SKILL.md` |

<!-- gitnexus:end -->

---
> Source: [allanschramm/local-model-autotuning](https://github.com/allanschramm/local-model-autotuning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
