## ai-dev-tasks

> Always record each chat turn to external memory via MCP.

After providing your final answer, call the MCP tool `record_chat_history` with a JSON object containing:

- `thread_id`: the active conversation thread identifier.
- `user_input`: the user\'s latest message.
- `system_output`: your final answer text.
- `llm_name`: the current model identifier (e.g. `"cursor-ai"`).
- `file_operations`: brief summary of any files or tools you touched this turn (`"none"` if nothing).

Example payload:
```json
{
  "thread_id": "<current_thread_id>",
  "user_input": "<last user message>",
  "system_output": "<your answer>",
  "llm_name": "<model name>",
  "file_operations": "<files/tools used or 'none'>"
}
```

Call `record_chat_history` immediately after finishing your answer so the turn is persisted in near real-time.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TheMonk2121) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
