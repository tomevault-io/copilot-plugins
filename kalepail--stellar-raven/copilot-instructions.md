## stellar-raven

> Canonical repository instructions for Codex, Claude Code, and other coding agents. Keep this file

# AGENTS.md — stellar-raven-codemode

Canonical repository instructions for Codex, Claude Code, and other coding agents. Keep this file
short, current, and operational. Put architecture, research, history, and task runbooks in the
linked docs and skills instead of accumulating them here.

## Project and source-of-truth map

This is a Cloudflare Workers MCP server exposing `search` and `execute` over Lumenloop, Stellar
Light/Scout, Stellar Docs (Algolia), and selected ecosystem skills. Model-authored JavaScript runs
in a networkless Dynamic Worker; host adapters own all service traffic, policy, and secrets.

- Read `PLAN.md` for current product scope and status, then `ARCHITECTURE.md` for the implemented
  request, catalog, scoring, sandbox, and auth design.
- Use `README.md` for connection and operator setup.
- Use `research/` for dated evidence and design context; it is not an instruction layer.
- Use `.agents/skills/<name>/SKILL.md` for repeatable task workflows. `.claude/skills` is the
  committed symlink to the same canonical directory.
- `CLAUDE.md` imports this file. Do not duplicate shared rules there.

## Commands and verification

- Install reproducibly: `npm ci`.
- Baseline validation for code changes: `npm run typecheck`, `npm test`, and `npm run build`.
- Run the narrowest relevant eval or maintenance command in addition to the baseline; the selected
  skill defines the exact gate for eval, drift, golden-truth, improvements, and observability work.
- Scan before committing: `npm run secrets:scan -- --tree`.
- Do not start a second Wrangler process. Use the existing Solo `dev` process for `npm run dev` and
  obtain its bound URL through Solo.
- Generated outputs are rebuilt by their `package.json` scripts, never edited by hand.

## Coordination

- The Solo project bound to this repo (currently project 49) is the control plane for dev
  processes, todos, scratchpads, timers, and long-running agents. Confirm scope with `whoami` and
  inspect `list_processes` before starting or replacing any of them.
- Independent adversarial review is a completion gate when requested: reviewer must differ from
  author, run to completion, and have every finding reconciled before finalization.
- Before spawning, inspect `list_agent_tools`. Add a permission-bypass flag only if the saved tool
  command lacks one; duplicating Codex `--yolo` breaks the spawn.
- Every Solo-spawned agent must run non-interactively, using the saved bypass flag or an added one,
  and its brief must require the same non-interactive mode for any descendants it spawns.
- Keep long-lived or shared fan-out in Solo so status and output remain visible to other agents.
- For a running service, use `get_process_ports` / `wait_for_bound_port` instead of guessing a URL.

## Model routing for repo-work fan-out

Use explicit model IDs and reasoning effort where the runtime supports it. Current evidence and
exact CLI mechanics live in `research/agent-model-roster.md`.

| Work | Preferred arm | Effort / review rule |
|---|---|---|
| Default hard implementation or analysis | `gpt-5.6-sol` | `high`; use `max` for genuinely frontier tasks |
| Balanced routine implementation | `gpt-5.6-terra` | `high`, with task-proportionate review |
| Mechanical sweeps and bounded first passes | `gpt-5.6-luna` | `medium` or `high`; require a stronger reviewer |
| Vendor-diverse adversarial review | `grok-4.5` | `high`; do not treat it as a taste-calibrated reviewer |
| User-facing design, API, code-quality, or copy review | Fable 5 or Opus 4.8 | Claude aliases `fable` (never `fable-5`) / `opus`; Sonnet 5 is acceptable for bounded review |

- Set Codex effort with `-c 'model_reasoning_effort="<effort>"'`; set Grok effort with
  `--reasoning-effort <effort>`. Do not rely on catalog defaults.
- Only choose repo-work models explicitly listed in the table above. Never use Haiku or an
  unlisted legacy model; catalog availability is not authorization.
- Sol/Terra `ultra` is a delegated multi-agent system, not another comparable reasoning depth;
  evaluate it as a separate arm.
- Public benchmark scores inform routing but do not become house cost/intelligence/taste ratings.
- If a selected arm misses the quality bar, rerun or redo the work with a stronger listed model
  without waiting for permission; cost is only a tie-breaker.
- Eval answering and judge models are separate measurement contracts controlled by `run-evals`.

## Hard rules

- **Forward-only:** prefer the best current design; do not add compatibility shims, dual formats,
  or deprecation paths merely to preserve deployed behavior. Deviations still need evidence.
- **The manifest is the exposed surface** (ADR-0003). Model code never owns endpoints, arguments,
  auth, or exposure. Never emit references to non-exposed operations or retired skills;
  `assertNoNonExposedRefs` is the build guard.
- Keep exact-match resolution for skill/tool IDs. Preserve service distinctions among soft-empty,
  error, and data responses.
- **Secrets stay host-side:** never print, commit, or expose credentials to the sandbox;
  `globalOutbound` remains `null`.
- The paid Lumenloop research trigger and its account-scoped read operations remain unexposed.
  Enabling them requires the exposure change, partner-detail persistence, budget gate, and dedup in
  one reviewed change.
- Partner-tier Lumenloop details are never committed. Inventory keeps name-only stubs and skill
  sync remains keyless.
- Any paid or side-effecting model operation requires host-side request-context approval,
  elicitation, and budget enforcement before it can ship.
- Algolia operator credentials are maintenance-only and never a runtime/sandbox surface. Any write
  needs a read-only A/B win, a general mechanism rather than per-query hacks, and the guardrails in
  `research/services/stellar-docs-algolia.md`.
- Evals produce evidence-backed upstream findings in `improvements/`; scores are instruments, not
  the final product.
- Retired sibling repos must not be referenced as live paths. Retained prior art is read-only under
  `eval/corpus/`; use `research/prior-art.md` for history.

## Task runbooks

Use the matching repo skill when the task triggers it:

- `truth-maintenance` — coordinate a full live-drift/eval/golden/improvements maintenance pass.
- `live-drift-resolution` — regenerate, classify, verify, and resolve live catalog drift.
- `run-evals` — select instruments, review verdicts, triage causes, and file findings.
- `improvements-pipeline` — maintain finding lifecycle, intake, probes, index, and upstream follow-up.
- `golden-truth` — change golden answers with provenance and explicit uncertainty.
- `cloudflare-observability-review` — investigate production logs, traces, telemetry, and Ray IDs.

Add durable repo-wide rules here only after recurring friction. Put specialized instructions in the
closest relevant skill or directory-level `AGENTS.md`.

## Definition of done

- The diff is scoped and preserves unrelated work in the dirty tree.
- Proportionate tests and required skill gates pass; failures are reported, not hidden.
- Generated artifacts came from scripts and secrets scanning passed where required.
- Requested independent reviews completed and every finding was reconciled.
- Documentation describes current behavior and links dated research instead of embedding history.

---
> Source: [kalepail/stellar-raven](https://github.com/kalepail/stellar-raven) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
