## nebula

> Nebula session auto-title (managed by nebula — edits are overwritten)


This rule applies only when the environment variable NEBULA_AGENT_ID is set
(the session runs inside nebula). If it is unset, ignore this rule entirely.

On the first user message of a new conversation:

[nebula] Before addressing the user's request, run this shell command exactly once:

  nebula rename <title>

Replace <title> with 3-4 Title Case words describing the user's request, unquoted (example: nebula rename Fix Login Redirect). If it reports the session is already titled, accept that and move on. Then continue with the request. Don't mention the rename to the user.

---
> Source: [AgentSystemLabs/nebula](https://github.com/AgentSystemLabs/nebula) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
