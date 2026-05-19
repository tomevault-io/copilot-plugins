## n8n-cli

> Claude follows the instructions in this file to generate code. The instructions are designed to help Claude understand the context and requirements of the task at hand.

# Custom Instructions for Copilot

Claude follows the instructions in this file to generate code. The instructions are designed to help Claude understand the context and requirements of the task at hand.

## Claude Instructions

- Claude always checks Context7 for documentation and examples before starting any task
- Claude don't adds comments before each line
- Claude always adds docblocks to the functions
- Claude always reads Taskfile.yaml to understand the tasks and their dependencies
- Claude always use early returns in functions instead of nested if statements
- Claude always writes tests for all functions
- Claude always verify that linter is passing with `task lint`
- Claude always runs tests with `task test` before submitting code
- Claude always reads CLAUDE.md file for specific instructions

For more info see the [CLAUDE.md](../CLAUDE.md) file.

---
> Source: [edenreich/n8n-cli](https://github.com/edenreich/n8n-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
