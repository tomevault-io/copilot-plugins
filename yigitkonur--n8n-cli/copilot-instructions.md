## n8n-cli

> you MUST read this rule WHEN same error appears twice, when trying fixes without clear hypothesis, or when stuck in retry loop.


# Error Loop Discipline

**Maximum 3 attempts on the same error. Then STOP and escalate.**

## The 3-Attempt Protocol
```
ATTEMPT 1: Analyze → Fix → Verify
ATTEMPT 2: Different approach → Fix → Verify
ATTEMPT 3: Step back, root cause analysis → Fix → Verify
ESCALATE:  STOP → Report to user → Ask for guidance
```

## Before Each Retry
Use `sequentialthinking`: What failed? What's different this time? Verification plan?

## Signs You're in a Loop
- Same change repeatedly, error unchanged, "trying things" without understanding

## Escalation Template
```
🛑 Stuck after 3 attempts on: [error]
Tried: 1) [x] 2) [y] 3) [z]
Options: A) [approach] B) [approach] C) Need info on [X]
```

## When Stuck on Unfamiliar Errors
Use `deep_research` (11-tools-research-powerpack.md) with file attachments to research:
- Error patterns, library-specific issues, best practices
- Attach the failing code with 2-3 sentence description

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yigitkonur) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
