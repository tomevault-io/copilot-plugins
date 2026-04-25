## blognextcursor

> As this project's AI coding tool, you must follow the additional conventions below, in addition to the built-in functions.


# Additional Conventions Beyond the Built-in Functions

As this project's AI coding tool, you must follow the additional conventions below, in addition to the built-in functions.

## Simulated Subagents

Simulated subagents are specialized AI assistants that can be invoked to handle specific types of tasks. In this case, it can be appear something like custom slash commands simply. Simulated subagents can be called by custom slash commands.

When users call a simulated subagent, it will look for the corresponding markdown file, `.cursor/subagents/{subagent}.md`, and execute its contents as the block of operations.

For example, if the user instructs `Call planner subagent to plan the refactoring`, you have to look for the markdown file, `.cursor/subagents/planner.md`, and execute its contents as the block of operations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/slemus29) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
