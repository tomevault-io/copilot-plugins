## synapsepythonclient

> [](){ #agent-reference-async }

# Agent

## API reference

[](){ #agent-reference-async }
::: synapseclient.models.Agent
    options:
        members:
            - register_async
            - get_async
            - start_session_async
            - get_session_async
            - prompt_async
            - get_chat_history
---
[](){ #agent-session-reference-async }
::: synapseclient.models.AgentSession
    options:
        members:
            - start_async
            - get_async
            - update_async
            - prompt_async
---
[](){ #agent-prompt-reference-async }
::: synapseclient.models.AgentPrompt
    options:
        inherited_members: true
        members:
            - send_job_and_wait_async
---

---
> Source: [Sage-Bionetworks/synapsePythonClient](https://github.com/Sage-Bionetworks/synapsePythonClient) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
