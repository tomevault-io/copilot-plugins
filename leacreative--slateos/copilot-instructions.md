## slateos

> You are working on a sealed PineTime thin-client watch + Android companion.

# Agent instructions (Slate / EvoTime)

You are working on a sealed PineTime thin-client watch + Android companion.
Hardware flashes are expensive. Recurring blank-face / OTA / Ambient bugs are
documented; **you must not re-learn them by flashing.**

## Mandatory before relevant work

1. Read [`docs/lessons-learned.md`](docs/lessons-learned.md) before changing
   **paint**, **OTA**, **BLE session / reconnect**, **power / Ambient**,
   **screen ownership**, or **notifications**.
2. Run through the **anti-pattern checklist** in that file before packaging DFU
   or claiming a fix.
3. Every handover that touches those areas MUST include a **Do not regress**
   block (see template below).

Also always: [`docs/capabilities.md`](docs/capabilities.md),
[`docs/issue-prompts-open.md`](docs/issue-prompts-open.md),
[`CLAUDE.md`](CLAUDE.md). Sub-apps → [`docs/subapp-rules.md`](docs/subapp-rules.md).
Drivers → [`docs/infinitime-parity.md`](docs/infinitime-parity.md) first.

## Do not regress (required in handovers)

When you finish work in a gated area, end the user-facing summary with:

```text
Do not regress:
- <invariant that must stay true, e.g. OTA sendable==0 while sent!=acked>
- <what not to delete/“clean up”, e.g. Ambient enter only when Core sleeping>
- <pointer pointer if any, e.g. OtaXferTest lock-step / planned Ambient gate>
```

If a cleanup removes a helper (`paint_local_now`, latch bypass, Ambient gate),
the handover must name **which invariant still holds and where** — or the
cleanup is incomplete.

## Version / install discipline

- Bump firmware `0.1.0-mN` and companion `versionCode`/`versionName` on every
  installable build.
- Before DFU that touches **OTA or Ambient/power display policy**, run
  `powershell -File scripts/run_invariant_tests.ps1` (must exit 0).
- Package `slate_dfu` when firmware changes; install companion on the Pixel
  when the APK version moves (see `.cursor/rules/companion-install.mdc`).

## Enforcement in this repo

| Mechanism | Location |
|---|---|
| Lessons + checklist | `docs/lessons-learned.md` |
| Always-on Cursor rule | `.cursor/rules/lessons-learned.mdc` |
| Path-scoped rules | `.cursor/rules/ota-lockstep.mdc`, `paint-ownership.mdc`, `ambient-power.mdc` |
| Session + edit hooks | `.cursor/hooks.json` |
| How enforcement works | `docs/agent-enforcement.md` |
| Planned invariant tests | `docs/invariant-tests-plan.md` |

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **SlateOS** (12130 symbols, 23653 relationships, 353 execution flows).

> Index stale? Run `node .gitnexus/run.cjs analyze --index-only` from the project root — it auto-selects an available runner. No `.gitnexus/run.cjs` yet? Bootstrap with `npx`, `bunx`, or `pnpm dlx` — e.g. `bunx gitnexus@latest analyze` (npm 11 npx crash; #1939).

## Always Do

- **MUST run impact analysis before editing.** Use `impact({target: "symbolName", direction: "upstream"})` (MCP) or `node .gitnexus/run.cjs impact "symbolName" --direction upstream --repo .` (CLI fallback); report callers, processes, and risk. Never substitute grep for graph analysis.
- **MUST analyze graph changes before committing.** Use `detect_changes({scope: "all"})` (MCP) or `node .gitnexus/run.cjs detect-changes --scope all --repo .` (CLI fallback). `partial: true` or `truncated: true` is not a clean check — a zero means unseen, not unaffected; re-run it. For regression review: `detect_changes({scope: "compare", base_ref: "master"})` or `node .gitnexus/run.cjs detect-changes --scope compare --base-ref "master" --repo .`.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- **MUST treat `risk: UNKNOWN` as unresolved, not as low.** An empty caller set is not evidence the symbol is unused — it can also mean the callers are not resolvable by the index (plain-object property access, dynamic dispatch, cross-language calls). `impact` pairs `UNKNOWN` with a `riskNote` saying so. Confirm with a text search before treating the symbol as safe to change or delete; do not proceed on the strength of a zero.
- When exploring unfamiliar code, use `query({search_query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `context({name: "symbolName"})`.
- For security review, `explain({target: "fileOrSymbol"})` lists taint findings (source→sink flows; needs `analyze --pdg`).

## Never Do

- NEVER edit a function, class, or method before MCP/CLI impact analysis.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis, and never read `UNKNOWN` as an all-clear — it means the walk could not answer, which is the one verdict that requires confirming by other means.
- NEVER rename symbols with find-and-replace — use `rename` which understands the call graph.
- NEVER commit before MCP/CLI graph change analysis.

## Resources

| Resource | Use for |
| --- | --- |
| `gitnexus://repo/SlateOS/context` | Codebase overview, check index freshness |
| `gitnexus://repo/SlateOS/clusters` | All functional areas |
| `gitnexus://repo/SlateOS/processes` | All execution flows |
| `gitnexus://repo/SlateOS/process/{name}` | Step-by-step execution trace |

## CLI

| Task | Read this skill file |
| --- | --- |
| Understand architecture / "How does X work?" | `.claude/skills/gitnexus-exploring/SKILL.md` |
| Blast radius / "What breaks if I change X?" | `.claude/skills/gitnexus-impact-analysis/SKILL.md` |
| Trace bugs / "Why is X failing?" | `.claude/skills/gitnexus-debugging/SKILL.md` |
| Rename / extract / split / refactor | `.claude/skills/gitnexus-refactoring/SKILL.md` |
| Tools, resources, schema reference | `.claude/skills/gitnexus-guide/SKILL.md` |
| Index, status, clean, wiki CLI commands | `.claude/skills/gitnexus-cli/SKILL.md` |

<!-- gitnexus:end -->

---
> Source: [LeaCreative/SlateOS](https://github.com/LeaCreative/SlateOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
