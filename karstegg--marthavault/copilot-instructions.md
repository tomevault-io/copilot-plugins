## marthavault

> To ensure new rules and workflows are correctly registered by the Windsurf IDE and are immediately available, the following manual creation process MUST be followed.


# Rule: Protocol for Creating New Rules and Workflows

## Principle: Ensure IDE Registration via Manual Creation

To ensure new rules and workflows are correctly registered by the Windsurf IDE and are immediately available, the following manual creation process MUST be followed.

### The Problem
Programmatic creation of rule (`.windsurf/rules/*.md`) and workflow (`.windsurf/workflows/*.md`) files by the AI agent can fail to be detected by the IDE's file watcher in real-time. This prevents them from appearing in the UI and being used in the current session without a restart.

### Mandatory Procedure
1.  **Agent's Responsibility:** When a new rule or workflow is required, the AI agent (Gemini) will NOT use file-writing tools (`write_to_file`, `run_command`, etc.) to create the file.
2.  **Agent's Output:** The agent will instead provide the complete and final content for the new file within a formatted markdown code block. The agent will also specify the required filename and path.
3.  **User's Responsibility:** The user will manually create the new file at the specified path and paste the content provided by the agent.

This procedure guarantees that all new automations are reliably loaded by the IDE.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/karstegg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
