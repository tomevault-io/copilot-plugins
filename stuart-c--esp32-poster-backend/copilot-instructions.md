## esp32-poster-backend

> * **Root Isolation:** All auxiliary worktrees must be created within the `.trees/` directory at the project root.

# Agent Workspace & Git Worktree Rules

## 1. Directory Structure
* **Root Isolation:** All auxiliary worktrees must be created within the `.trees/` directory at the project root.
* **Pathing:** The standard format for a worktree path is `.trees/agent-{task-id}`.
* **Gitignore:** Ensure `.trees/` is added to the global `.gitignore` to prevent recursive indexing of the entire repository history.

## 2. Worktree Lifecycle
* **Creation:** Before starting a high-context task (e.g., refactoring or multi-file features), the agent should check if a dedicated worktree exists. If not, create one:
    `git worktree add -b <branch-name> .trees/<task-id> main`
* **Environment Setup:** After creation, the agent must initialize the environment (e.g., `npm install` or `uv sync`) within that specific directory before executing code.
* **Cleanup:** Upon successful PR merge or task completion (e.g., GitHub Action verified), the agent must prompt the human user (who acts as the Manager) to approve a terminal command running `git worktree remove .trees/<task-id>`.

## 3. Concurrency & Locking
* **Port Management:** When running local servers, agents should note standard base ports (e.g., Vite: `5173`, FastAPI/Flask: `8000`, LocalStack: `4566`) and must use dynamic port assignment relative to these to avoid `EADDRINUSE` conflicts with the main worktree or other agents.
* **Database Isolation:** If the project uses a local SQLite or dev database, agents must point to a task-specific DB instance within their `.trees/<task-id>` folder.

## 4. State Persistence
* **No Stashing:** Do not use `git stash` within a worktree. Commit work-in-progress to the task branch with a `WIP: <description>` message to ensure the Manager can recover state if the agent crashes.

## 5. AGENTS files
* Check for the presence of AGENTS.md files in the project workspace
* There may be additional AGENTS.md in sub-folders with additional specific instructions that are related to only that part of the codebase.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stuart-c)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/stuart-c)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
