## sns-s-s

> SNS-S-S is a formal research instrument for Solar-Nano-Sphere swarm dynamics, explicit energy-chain modeling, asteroid-resource intelligence, and evidence-driven research execution. It is not flight software and must never imply hardware readiness.

# AGENTS.md: SNS-S-S Stable Law

SNS-S-S is a formal research instrument for Solar-Nano-Sphere swarm dynamics, explicit energy-chain modeling, asteroid-resource intelligence, and evidence-driven research execution. It is not flight software and must never imply hardware readiness.

## Authority order

1. platform safety and explicit human instruction;
2. this stable law;
3. the selected active contract in `automation/contracts/`;
4. the active issue and PR acceptance slice;
5. the active quest record;
6. accepted monthly governance;
7. accepted weekly evidence;
8. `memory/mem_log_short.md`;
9. historical logs and older plans.

No lower layer silently overrides a higher layer.

## Loop routing

Select exactly one contract before acting:

- daily implementation: `automation/contracts/daily-research-operator.v1.md`;
- weekly evidence: `automation/contracts/weekly-evidence-synthesis.v1.md`;
- monthly governance: `automation/contracts/monthly-governance.v1.md`;
- system audit: `automation/contracts/system-audit.v1.md`.

Read only the contract’s required inputs plus files needed for the acceptance slice. Every trigger ends with one immutable receipt under `automation/runs/**` and one explicit terminal state.

## Scientific and engineering invariants

1. Prefer explicit dataclasses, enums, units, and small inspectable functions.
2. Preserve units in names such as `_W`, `_Wh`, `_s`, `_K`, and `_m2`.
3. Resolve harvested energy into direct use, storage, delivery, or curtailment.
4. Separate score from confidence, especially in ARCI.
5. Keep mission assumptions configurable rather than hidden in code.
6. Preserve legacy compatibility intentionally; new work follows the Summer 2026 ontology.
7. Simulation PASS means declared model criteria passed, not space qualification.
8. Test every new behavior and record limitations, uncertainty, and falsifiers.
9. Keep active quests artifact-first and bounded to 1–8 entries.
10. Do not alter scientific claims through automation-infrastructure work.

## Shared-state law

`automation/state_ownership.json` is normative. Daily loops execute within accepted scope. Weekly loops normalize evidence and propose changes. Monthly governance owns queue-wide decisions, consolidated beliefs, stale-work resolution, and the canonical next move. Audit loops observe and recommend.

Raw run, evidence, belief, and quest-action events are immutable. Corrections are linked new events. Shared Markdown append history is not a required write surface. Generated logs derive from receipts.

## Concurrency and PR law

Record source commit and state hashes before work. Recheck governance, owned state, and PR ownership immediately before publication. One PR owns one quest acceptance slice. Continue the valid owner, split expanding work, and classify stale or superseded branches explicitly. Automatic merge remains conservative.

## Terminal states

Use only:

- `DONE`, `DONE_WITH_LIMITATIONS`;
- `BLOCKED_ENVIRONMENT`, `BLOCKED_MISSING_EVIDENCE`, `BLOCKED_CONFLICT`;
- `VERIFICATION_FAILED`;
- `NEEDS_SCIENTIFIC_DECISION`, `NEEDS_GOVERNANCE_REVIEW`, `NEEDS_APPROVAL`.

A precise blocker is a valid outcome. Do not continue merely to avoid reporting one.

## Definition of done

An acceptance slice is complete only when its artifact exists, checks are evidenced, assumptions and limitations are explicit, ownership is current, semantic repository validation passes, the receipt is immutable, and the next action is one concrete transition.

---
> Source: [xxx3r/SNS-S-S](https://github.com/xxx3r/SNS-S-S) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
