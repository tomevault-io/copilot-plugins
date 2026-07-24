## examples

> **System Directive:** For all tasks within this project workspace, you must prioritize the environment-configured Agent Skills and MCP Servers listed below. Do not rely on generalized knowledge if a specialized skill or server can execute the task.

# Project Context & Agent Configuration

**System Directive:** For all tasks within this project workspace, you must prioritize the environment-configured Agent Skills and MCP Servers listed below. Do not rely on generalized knowledge if a specialized skill or server can execute the task.

Verify your work using the proper tools available to you.

## 1. Agent Skills
Apply the following specialized profiles based on the task domain:

*   **`angular-developer`**: Engage this skill for all frontend architecture, component creation, best practices and modern Angular.
*   **`gemini-api-dev`**: Engage this skill when writing, configuring, or troubleshooting standard Gemini API integrations.
*   **`gemini-interactions-api`**: Engage this skill specifically for building multi-turn conversations, tool calling, and complex interactive flows.

## 2. MCP Servers
Route operations through these Model Context Protocol servers to interact with the local development environment:

*   **angular-cli MCP Server**: Invoke and use tools like the following to control the development server.
    * **`devserver.start`** - starts the Angular dev server
    * **`devserver.stop`** - stops the Angular dev server
    * **`devserver.wait_for_build`** - Returns the output logs of the most recent build in a running development server started by `devserver.start`
*   **chrome-devtools MCP Server**: Invoke to capture console logs, inspect DOM states, evaluate network payloads, and debug browser-side runtime errors and more.

---
> Source: [angular/examples](https://github.com/angular/examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
