## llm-consilium-skill

> Use on explicit request for a multi-LLM/model council to collect independent candidate opinions, compare claims, preserve dissent, and produce a bounded synthesis without treating consensus as truth.


# LLM Consilium

## Overview

Use this skill to run a deliberately triggered multi-model council. The goal is not to make agents debate indefinitely; the goal is to obtain independent opinions, identify common ground and disagreements, verify important claims when needed, and synthesize a better final answer.

This skill must **not** be auto-routed for ordinary questions. Use only when the user explicitly asks for a consilium/council or asks to gather independent model opinions.

Default user-facing output: concise, practical, and in the user's language.

## Operating Principles

1. **Independent first pass.** Candidates answer before seeing each other.
2. **Consensus is not truth.** Agreement is a signal, not proof.
3. **Preserve dissent.** Minority opinions can be right.
4. **Orchestrator is technical.** The current agent/session manages prompts, files, process execution, claim extraction, and synthesis, but should not judge its own answer as winner.
5. **Claim-level comparison beats answer-level comparison.** Extract atomic claims/recommendations and compare them.
6. **Use tools for facts.** Verify math, current facts, code, files, and external claims with tools/sources/tests where practical.
7. **No endless debate.** Debate only contested high-impact claims, and only in full/high-stakes mode.

## Candidate Configuration

The repository is agent-agnostic and model-agnostic. Candidate IDs, model names, commands, and route-specific details belong in config, not in prose.

- Template config: `templates/llm-consilium.json`.
- Deployed config: choose your own path with `LLM_CONSILIUM_CONFIG`, `--config`, or installer options.
- The included config is a safe placeholder and must be customized with your local model/agent CLIs before real non-dry-run use.

A candidate command may use tokens:

- `{PROMPT}` — full prompt as an argv argument.
- `{WORKSPACE}` — isolated candidate workspace path.
- `{PROMPT_FILE}` — path to the candidate prompt file.

Prompt transports:

- `argv`: replace `{PROMPT}` in command argv.
- `stdin`: send the prompt to process stdin.
- `file`: command reads `{PROMPT_FILE}`.

## Modes

### Simplified / fast mode

Use when the user asks for a lightweight council or does not request a full evidence pass.

Pipeline:

1. Prepare one shared prompt.
2. Run configured candidates independently with the same core prompt.
3. Compare outputs at claim level.
4. Return consensus, disagreements, risks, weak claims, confidence, and synthesis.

### Full mode

Use when the user explicitly asks for a full council, when the decision is high-stakes, or when factual accuracy matters more than speed.

Runner `--mode full` means an expanded configured candidate set and stricter artifact discipline. Real evidence checking, judge passes, and targeted debate are explicit additional layers.

Full-mode additions:

- save raw outputs and logs;
- anonymize candidate outputs for reviewer/judge passes when used;
- separate model diversity from role diversity;
- verify important factual claims with tools/sources/tests;
- show minority views and unresolved disagreements;
- explain confidence from agreement + evidence + risk, not self-reported confidence.

## Candidate Prompt Template

```text
You are an independent participant in a multi-model council. Reply in the user's language. Be concise and practical.

Task:
{{QUESTION}}

Do not guess other models' opinions. Do not refer to consensus. Provide an independent position.

Return this structure:
## Position
...

## Main arguments
- ...

## Risks / caveats
- ...

## Confidence
low|medium|high — why

## What could change the conclusion
- ...
```

## Output Template

```markdown
## Consilium: synthesis

**Verdict:** ...
**Confidence:** low|medium|high

**Consensus:**
- ...

**Disagreements:**
- ...

**Strongest arguments:**
- ...

**Risks / weak claims:**
- ...

**Synthesis:**
...
```

## Runner

```bash
llm-consilium-run <slug> --question-file /path/to/question.md --mode fast
llm-consilium-run <slug> --question-file /path/to/question.md --mode full
llm-consilium-synthesize /path/to/artifacts/<run-dir>
```

Useful dry run:

```bash
llm-consilium-run smoke --question 'Route check' --mode fast --dry-run
```

The runner creates an artifact directory containing prompt, per-candidate workspaces, raw outputs, logs, manifest, and analysis files.

Deterministic synthesis writes:

- `analysis/claims.json`
- `analysis/repetition_report.json`
- `analysis/stance_matrix.json` (compatibility copy; not a full contradiction graph)
- `analysis/evidence_ledger.json`
- `analysis/summary.md`
- `final.md`

This deterministic stage is **model-consistency-only**. It does not prove factual truth.

## Evidence Labels

Use labels honestly:

- `single-model-only`
- `cross-model-repeated`
- `all-models-repeated`
- `model-consistency-only`
- `verified-after-tool-check` only after a real external check
- `contradicted-after-tool-check` only after a real external check
- `unresolved`

## Optional Review / Debate Layers

- Anonymous peer review: useful when outputs diverge materially.
- Judge pass: useful for high-stakes decisions; anonymize model identities.
- Targeted debate: only for contested high-impact claims.
- Evidence pass: required before calling factual claims verified.

Do not make these layers mandatory for every fast run.

## Common Pitfalls

1. **Auto-triggering consilium.** This skill is explicit-trigger only.
2. **Majority vote as truth.** Consensus is a signal; verify important claims.
3. **Model identity bias.** Anonymize outputs for review/judging.
4. **Role/model confusion.** Do not assign fixed roles to models in the first candidate layer.
5. **Debating everything.** Debate only contested high-impact claims.
6. **Inventing model IDs.** Configure and preflight actual local routes.
7. **Overstating deterministic synthesis.** Repetition is not verification.
8. **Dumping too much to chat.** Keep the user-facing summary compact; keep raw detail in artifacts.

## Verification Checklist

- [ ] Explicit user request or high-stakes reason for consilium.
- [ ] Mode chosen: fast or full.
- [ ] Candidate prompts use the same core prompt for the first pass.
- [ ] Raw outputs/logs saved when needed.
- [ ] Failed/empty/no-final outputs excluded or marked.
- [ ] Consensus and disagreements are claim-level.
- [ ] Important factual/tool-checkable claims verified where practical.
- [ ] Minority views and unresolved disagreements visible.
- [ ] Confidence explained from agreement + evidence + risk.
- [ ] Final answer concise; raw artifacts kept separately.

---
> Source: [mingazhev/llm-consilium-skill](https://github.com/mingazhev/llm-consilium-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
