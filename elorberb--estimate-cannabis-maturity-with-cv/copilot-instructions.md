## estimate-cannabis-maturity-with-cv

> Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes


# Systematic Debugging

**Core principle:** ALWAYS find root cause before attempting fixes. Symptom fixes are failure.

**Iron Law:** NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST.

## The Four Phases (complete in order)

1. **Root Cause:** Read error messages; reproduce consistently; check recent changes; gather evidence at component boundaries; trace data flow to source.
2. **Pattern Analysis:** Find working examples; compare to references; list differences; understand dependencies.
3. **Hypothesis and Testing:** Form one clear hypothesis; make smallest change to test it; verify before continuing.
4. **Implementation:** Create failing test case first; implement single fix; verify. If 3+ fixes failed, question the architecture with your human partner — do not try more fixes.

## Red Flags — STOP and Follow Process

"Quick fix for now"; "Just try changing X"; proposing solutions before tracing; "One more fix attempt" after 2+ failures; each fix reveals new problem elsewhere.

**If 3+ fixes failed:** Stop. Question whether the pattern/architecture is sound; discuss before more fixes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/elorberb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
