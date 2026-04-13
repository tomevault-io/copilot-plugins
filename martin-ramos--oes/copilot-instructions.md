## oes

> Full spec: `.opencode/OES.md`

# OES — Codex

Full spec: `.opencode/OES.md`

@std: cc[fn-small,names-clear,no-dup,no-mixed-resp] solid[srp,ocp?,dip?] sec[no-secrets,validate-input]
@perf: O(n) declare when non-trivial | no-recompute
@rel:  err-explicit | invariants | state-consistent
@end:  review-final + EQI[0-100]

## Engram (Mandatory)

start: mem_context
decisions: mem_save (type: decision/architecture)
end: mem_session_summary

Store only: architectural decisions, pattern choices, trade-offs, constraints.
Never store: code, diffs, ephemeral reasoning.

## Modes

active: standard
- performance-strict → @perf+ | O(n) always | hot-paths
- high-reliability   → @rel+  | err-explicit | invariants

## Commands

/work $task     → classify → skills? → exec @std → @end
/feature $task  → explore→spec→design→plan→impl→verify → @end
/bug $task      → root-cause → fix-minimal → verify → @end
/refactor $task → no-behavior-change → @std → @end
/review $code   → eval[@std,@perf,@rel] → EQI[0-100]
/sdd $design    → phases[6] → engram_remember → @end
/review-pr $pr  → compile-first | diff+commits+scope+patterns → verdict[APPROVED|OBS|REJECTED]

## Architecture

- Skills:    `.opencode/skills/`
- Modes:     `.opencode/modes.md`
- Infra:     `.opencode/infrastructure/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/martin-ramos)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/martin-ramos)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
