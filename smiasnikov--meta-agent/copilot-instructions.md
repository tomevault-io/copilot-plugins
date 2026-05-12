## meta-agent

> - **FPF skill is required** for all work inside meta-agent artifacts.

# Meta-Agent Rules (Operational)

## Mandatory skill
- **FPF skill is required** for all work inside meta-agent artifacts.

## Core FPF rules
- Enforce A.7: Object ≠ Description ≠ Carrier; Role ≠ Work; MethodDescription ≠ Work.
- Enforce A.2.2: Capability ≠ Method. Capability is a dispositional property of a System, not a recipe.
- Every claim that affects capabilities must have Evidence (A.10).
- Evidence must be stored as separate records with F-G-R + decay (B.3/B.3.4).

## Boundary rules (A.6.B)
- No applied-domain knowledge inside meta-agent artifacts.
- Applied content lives in applied agent folders or in meta-agent experiments/hypotheses as test data only.
- Applied agent naming requires explicit human approval (HumanApprover).

## Change control (DRR per E.9)
- Decision log follows canonical DRR format (E.9:4): Problem frame, Decision, Rationale, Consequences.
- Decision log must be updated on any structural change.
- Lightweight variant (CC-DRR.5) permitted for non-semantic edits.
- Capabilities remain provisional until evidence-backed.

## Process
- Work in small steps, show file content before writing.
- Maintain linkage: Hypothesis → Experiment → Evidence → Capability.
- Evidence audit (A.10/B.3) is a standard step in experiments; can be smoke-level or full.
- Capability status upgrades require evidence audit pass (at least smoke-level).
- Evidence naming must include context prefix (MA.EV-NNN for meta-agent, ST.EV-NNN for Strategator, etc.).

---
> Source: [smiasnikov/meta-agent](https://github.com/smiasnikov/meta-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
