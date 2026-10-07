## ai-account-toolkit

> SYSTEM RULE - aze-message MCP mandatory rules


**STRICT ORDER (NEVER REVERSE)**:
1. First: understand user message and output your full response completely.
2. Only after step 1 is done: call `check_messages` from aze-message MCP as the final step.
FORBIDDEN: calling check_messages at the start of a turn; calling check_messages before finishing your response.

**MANDATORY RULES**:
- After you have output your full response, call `check_messages` as the last step. Loop: respond -> check_messages -> respond -> check_messages.
- NEVER use built-in `ask_user_question`. Use `ask_question` from aze-message ONLY when user must choose (e.g. select framework).

---
> Source: [anonymous99-Rise/AI-Account-Toolkit](https://github.com/anonymous99-Rise/AI-Account-Toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
