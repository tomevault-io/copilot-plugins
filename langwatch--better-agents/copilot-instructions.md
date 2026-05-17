## correction-response

> Respond to corrections by creating or updating rules


# Correction Response Rule

**When corrected for doing something wrong, evaluate if it reveals a pattern that should become a rule.** Only create/update rules for systemic issues, not one-off instructions.

## Process:
1. **Acknowledge the correction** and thank the user
2. **Ask for clarification** if the correction seems like a one-off instruction vs. a principle
3. **Identify the root cause** of the mistake
4. **Create or update a rule** only for patterns that should be permanently avoided
5. **Apply the rule** immediately to the current work if created
6. **Document the change** clearly

## Examples of Rule-Worthy Corrections:
- "no any in typescript" → Create typing rules (systemic principle)
- "rule too long" → Update rule formatting guidelines (process improvement)
- "use pnpm not npm" → Update tooling preferences (workflow consistency)

## Examples of One-Off Instructions (Don't Make Rules):
- "skip the test" → Just skip this one test, don't change testing workflow
- "use red color here" → Specific UI choice, not a principle
- "name this variable foo" → Specific naming, not a pattern

## When to Ask for Clarification:
- If the correction seems situational or temporary
- If it contradicts existing patterns/principles
- If you're unsure if it's a one-off or systemic issue

This ensures meaningful rule creation while respecting situational instructions.

---
> Source: [langwatch/better-agents](https://github.com/langwatch/better-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
