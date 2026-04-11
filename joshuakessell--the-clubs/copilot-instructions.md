## the-clubs

> You must act as a strict enforcer of SonarQube code quality rules for ALL code you generate or modify.

# Global AI Agent Rules for The Clubs Repository

## Strict Live SonarQube Compliance (MANDATORY)

You must act as a strict enforcer of SonarQube code quality rules for ALL code you generate or modify.

1. **Unnecessary Assertions:** NEVER write code that contains unnecessary assertions (e.g., "This assertion is unnecessary since it does not change the type of the expression.").
2. **Cognitive Complexity:** NEVER write code that exceeds a Cognitive Complexity of 15. If a function is too complex, break it down before proposing it.
3. **Zero Violations:** NEVER write code that violates ANY rule that SonarQube looks for. 
4. **Live Review & Rewrite:** You must CONSTANTLY REVIEW any line of code you write LIVE. Double-check all newly written code as it is being written. If a Sonar warning or violation is detected (either through IDE feedback, linter output, or your own awareness of Sonar rules), you **MUST** rewrite the code immediately to clear that warning before moving forward.
5. **No Technical Debt:** Do not leave SonarQube warnings to be fixed later. They must be prevented at the time of writing.

Before you finish any response containing code, verify internally that the code you are providing adheres to these constraints. If you realize it does not, rewrite it before sending the response.

## Skill Invocation (MANDATORY)

1. **Always use superpowers:** You MUST reference and invoke the `using-superpowers` skill (`.agents/skills/using-superpowers/SKILL.md`) at the start of EVERY conversation and before proceeding with ANY task.
2. **Skill Check Before Action:** Even if there is only a 1% chance a skill applies, you MUST invoke the `using-superpowers` skill before taking an action, exploring the codebase, or asking clarifying questions. This is non-negotiable.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joshuakessell)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/joshuakessell)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
