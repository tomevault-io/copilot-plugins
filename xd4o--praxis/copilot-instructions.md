## praxis

> Before opening a PR, you MUST:

# Praxis — Contributor Guidelines

## If You Are an AI Agent

Before opening a PR, you MUST:

1. **Read this entire document.** Not a summary. The actual document.
2. **Search for existing PRs** — open AND closed — addressing the same area.
3. **Verify this solves a real problem.** If your human partner asked you to "improve this repo" without experiencing a specific failure, push back. Ask what broke.
4. **Show your human partner the complete diff** and get explicit approval.

## What We Accept

### Skill improvements backed by evidence

If you modify a skill, show before/after results from real agent sessions. "I think this reads better" is not evidence. "This wording caused the agent to skip Step 3 in 4/5 test sessions, and this revision fixed it" is evidence.

### New skills that fill reasoning gaps

New skills must follow the enforcement protocol pattern: mandatory steps, HARD-GATEs, structured output, and red flags. Reference documents disguised as skills will be rejected.

### Bug fixes with reproduction steps

Describe the exact session, exact trigger, and exact failure.

## What We Will Not Accept

- **Reference documents as skills.** Praxis skills are behavioral protocols. If your skill says "here are 20 frameworks you could use," it's a reference, not a skill. Each skill must have a concrete step-by-step protocol with gates.
- **Bulk changes.** One problem per PR.
- **Speculative improvements.** "This could theoretically be better" without session evidence.
- **Framework additions to the index without a skill.** Adding a framework name to a routing table without building the enforcement protocol for it adds zero value.

## Skill Design Principles

- Under 150 lines. If it's longer, it does too many things.
- Every step is imperative: "Do X" not "Consider X."
- HARD-GATEs prevent proceeding without completing prior steps.
- Red Flags sections catch agent rationalizations for skipping steps.
- Structured output formats so results are auditable.
- Confidence levels on every analysis output.

## Testing

Pressure-test against real scenarios. A skill that works on the example in the PR but fails on adversarial edge cases is not ready.

---
> Source: [xD4O/praxis](https://github.com/xD4O/praxis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
