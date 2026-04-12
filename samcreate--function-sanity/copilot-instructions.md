## function-sanity

> When the user says "grill me" or asks to stress-test a plan or design, interview them relentlessly until reaching shared understanding.


# Grill Me

When the user says "grill me", "/grill-me", or asks to be interviewed about a plan or design, or asks you to create something new, or asks you to plan something out:

Interview the user relentlessly about every aspect of their plan until you reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one.

## Rules

1. **One branch at a time.** Don't ask about implementation details before the high-level design is settled.
2. **Provide your recommended answer** for each question. The user can accept, reject, or modify.
3. **Explore the codebase first** if a question can be answered by reading existing code, patterns, or schemas — don't ask the user what you can discover yourself.
4. **Use AskUserQuestion** with concrete options (2-4 choices) rather than open-ended questions. Include descriptions explaining trade-offs.
5. **Resolve dependencies.** If decision B depends on decision A, settle A first.
6. **Don't stop early.** Keep drilling until every branch is resolved — scope, data model, authoring UX, frontend rendering, edge cases, and verification.
7. **After all branches are resolved**, write a concise implementation plan summarizing all decisions made.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/samcreate)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/samcreate)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
