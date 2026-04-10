## you-have-to-deliver

> - Always run `git lfs install` and `git lfs pull` from the repo root before starting any task

# AGENTS.md

## Instructions for ChatGPT / Codex agent
- Always run `git lfs install` and `git lfs pull` from the repo root before starting any task
- Set the remote url and use the `GITHUB_TOKEN` environment variable for authentication:
  ```bash
  git remote set-url origin https://${GITHUB_TOKEN}@github.com/ahmadfarhan1981/you-have-to-deliver.git
  git lfs pull origin
  ```
  
## Codebase description
1. This codebase is software development company simulation game written in rust with legion ECS and tauri with a Svelte frontend.

## General Directions
Always Use the following guidelines to solve tasks:
1. If you have access to MCP servers. Always use `sequential thinking` to think through the steps required to execute the task.
2. If you don't, try to think off the steps required before executing the tasks. Review the task to make sure they make sense, then only proceed.
3. Verify any assumption by checking the files. Do not continue working based on unverified assumption
4. Test often. Run code incrementally if possible instead of in big chunks.
5. Add test cases related to the change as appropriate.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ahmadfarhan1981)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ahmadfarhan1981)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
