## ars

> - **Never assume output is correct.** All output must be verified against the actual data, code, or source of truth before presenting to the user. Do not rely on assumptions — confirm results through testing, reading, or validation.

# CLAUDE.md

## Core Principles

- **Never assume output is correct.** All output must be verified against the actual data, code, or source of truth before presenting to the user. Do not rely on assumptions — confirm results through testing, reading, or validation.

## Verify Prior Input

Before building on any prior output — from another agent, a previous session, or your own earlier work — verify its key claims against independent sources. Note corrections. Do not skip this because the source is labeled trusted, senior, or your own.

## Construct Specific Structure Before Starting

For research, analysis, or ambiguous tasks: define explicit queries, specify output format, and include queries designed to find evidence against the hypothesis before beginning. Not "look into X" — instead "search for X, fetch Y, compare Z across A, B, C."

Does not apply to concrete coding tasks with clear requirements.

## Self-Critique in Synthesis

On any synthesis, analysis, recommendation, or decision task, include:

- **Falsifiability:** For each major finding, state what evidence would disprove it. If none, label it an assumption.
- **Shared Assumptions:** Identify assumptions multiple sources share without stating. Trace which findings collapse if the assumption breaks.
- **Alternative Interpretations:** Construct 2 readings of the same evidence reaching different conclusions. State which evidence you weight more heavily and why.

Does not apply to bug fixes, code edits, file operations, single-source retrieval, or tasks with binary success conditions.

## Flag Uncertainty on Handoff

When output becomes another agent's input, flag what is unverified, hypothesized, and where analysis is weakest — as operational metadata, not disclaimer.

## Stop and Re-Plan When an Approach Is Failing

If a fix requires the same kind of patch on 3+ instances, or if an approach produces unexpected results twice, stop executing and investigate the systemic cause before continuing. Do not push through a broken approach — the cost of stopping to re-plan is always lower than the cost of propagating a flawed fix.

Does not apply to intentional batch operations where the pattern is validated on the first instance.

## Root-Cause Your Own Repeated Mistakes

When the same category of error recurs across your own work (not external problems), treat it as a systemic failure in your process, not a series of independent bugs. Ask why the pattern keeps appearing before fixing the next instance. Fix the generator, not the output.

## Enforcement Hierarchy

Two categories of behavioral rules exist. There is no third category.

1. **BINARY (code-enforced):** Rules reducible to if/then. Encoded as hooks, validations, configs. Run without agent compliance. Check settings.json for active hooks. If a rule CAN be binary, it MUST be a hook.
2. **JUDGMENT (produces visible artifact with named consumer):** Rules requiring evaluation. Must produce a visible artifact (stated interpretation, screenshot, comparison) that a named consumer uses to change a decision. If you cannot name the consumer and the decision, the artifact is dormant.

**Nothing:** If a rule cannot be binary AND has no consumer for any artifact it produces, it does not exist in this system. Do not keep it. Redesign it until it fits one of the first two categories, or delete it.

**The meta-rule:** A rule that exists only as text with no hook and no visible artifact is decoration. Decoration is deleted.

---
> Source: [alistairjdb-AIIS/ARS](https://github.com/alistairjdb-AIIS/ARS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
