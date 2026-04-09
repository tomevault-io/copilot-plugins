## cloudflare-mono-repo

> <!-- nx configuration start-->

<!-- nx configuration start-->
<!-- Leave the start & end comments to automatically receive updates. -->

# General Guidelines for working with Nx

- When running tasks (for example build, lint, test, e2e, etc.), always prefer running the task through `nx` (i.e. `nx run`, `nx run-many`, `nx affected`) instead of using the underlying tooling directly
- You have access to the Nx MCP server and its tools, use them to help the user
- When answering questions about the repository, use the `nx_workspace` tool first to gain an understanding of the workspace architecture where applicable.
- When working in individual projects, use the `nx_project_details` mcp tool to analyze and understand the specific project structure and dependencies
- For questions around nx configuration, best practices or if you're unsure, use the `nx_docs` tool to get relevant, up-to-date docs. Always use this instead of assuming things about nx configuration
- If the user needs help with an Nx configuration or project graph error, use the `nx_workspace` tool to get any errors
- For Nx plugin best practices, check `node_modules/@nx/<plugin>/PLUGIN.md`. Not all plugins have this file - proceed without it if unavailable.

<!-- nx configuration end-->

# Hello Agents

- The folder [LLM](./.llm) is your workspace for collaborative coding, session export (i.e chat transcription backup/console log), shared scripts, and scratchpad (USE this folder to maintain an organized front with the other agents and myself)
- The canonical plan lives in `.llm/docs/` and all documentation updates should go there.
- You are responsible for maintaining the documentations, notes, PLAN.md, TODO.md, and CHANGELOG.md in `.llm/` (NON NEGOTIABLE).
- Each agent has a specific purpose and set of tools to help you with different tasks.
- Use the appropriate agent based on the task at hand to get the best assistance.
- you all have a shared responsibility to keep each other updated on the progress of tasks, any changes in plans, and any issues that may arise. Communication is key to our success as a team.
- Remember to document your work thoroughly in the [LLM](./.llm) folder, (treat FOLDER like your office) including any decisions made, code changes, and important information that may be useful for future reference.
- Please ensure that you follow best practices for code quality, testing, and documentation. This will help maintain the integrity of the codebase and make it easier for future developers to work with.

[agents.prompt](.llm/agents.prompt.yml) 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rainbowkillah)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rainbowkillah)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
