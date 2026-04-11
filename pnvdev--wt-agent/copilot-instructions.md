## wt-agent

> `wt-agent` is a Command Line Interface (CLI) tool written in 100% Node.js and TypeScript, specifically designed to be executed by autonomous AI agents (like `gemini-cli`). Its purpose is to create and manage Git worktrees deterministically, allowing agents to develop multiple features in parallel, isolated environments without running into state conflicts.

# wt-agent - Agentic Git Worktree CLI

## Project Overview
`wt-agent` is a Command Line Interface (CLI) tool written in 100% Node.js and TypeScript, specifically designed to be executed by autonomous AI agents (like `gemini-cli`). Its purpose is to create and manage Git worktrees deterministically, allowing agents to develop multiple features in parallel, isolated environments without running into state conflicts.

### Key Architectural Decisions:
*   **Agent-First Output:** The CLI defaults to structured JSON output (`{ success: boolean, data?: any, error?: { code, message } }`), avoiding the need for fragile regex parsing of standard command outputs.
*   **Decoupled Architecture:** 
    *   **Presentation Layer (`src/index.ts`):** Uses `commander` to parse inputs and inject configuration.
    *   **Service Layer (`src/services/git.ts`):** Encapsulates all Git logic using standard `child_process.execSync` against the host's `git` binary, ensuring local credentials/helpers are used without heavy native Node dependencies.
    *   **Utility Layer (`src/utils/output.ts`):** Standardizes stdout/stderr responses.
*   **Idempotency & Determinism:** Paths are predictable (`.worktrees/<feature-name>`), and the tool handles existing worktrees or branches safely to support retry loops from agents.

## Building and Running

The project uses a standard Node.js and TypeScript toolchain.

*   **Install dependencies:**
    ```bash
    npm install
    ```
*   **Build the project (Compile TypeScript to `dist/`):**
    ```bash
    npm run build
    ```
*   **Run the CLI (Development):**
    ```bash
    npm start -- [command] [options]
    ```
*   **Run the CLI (Production/Compiled):**
    ```bash
    npx @pnvdev/wt-agent [command] [options]
    ```

### Available Commands:
*   `create <feature>`: Creates a new worktree for a feature (optionally specifying `-b <base_branch>`).
*   `list`: Lists all active feature worktrees and their status.
*   `switch <feature>`: Returns the deterministic local path to a specific feature's worktree.
*   `remove <feature>`: Removes a worktree safely. Requires `-f` or `--force` if the worktree contains uncommitted changes.
*   `sync <feature>`: Fetches and rebases the worktree against its remote origin base.
*   `doctor`: Analyzes the environment health, checks for orphaned worktrees, and verifies the Git repository.

## Development Conventions

*   **TypeScript Configuration:** Compiles to CommonJS/ES2022 to optimize for CLI execution without experimental ESM flags (`tsconfig.json`).
*   **Error Handling:** Exceptions in the Service layer (`src/services/git.ts`) should throw standard `Error` objects with descriptive messages. The Presentation layer (`src/index.ts`) catches these and delegates to `emitOutput` to format them into structured JSON errors.
*   **No Interactive Prompts:** Because this tool is meant to be run as a sub-process by an agent, no part of the code should ever wait for TTY/interactive input from the user.
*   **Human-Readable Mode:** While JSON is the default, all commands support an optional `--human` flag that will output standard console messages (`\x1b[32m[SUCCESS]\x1b[0m` or `\x1b[31m[ERROR]\x1b[0m`) for manual developer debugging.

## Project Roadmap

### Completed Tasks
* [x] Project scaffolding (Node.js + TypeScript).
* [x] Implementation of CLI router using `commander`.
* [x] Implementation of `GitService` to manage worktrees.
* [x] Standardized output JSON for agent readability.
* [x] Implemented core commands: `create`, `list`, `switch`, `remove`, `sync`, and `doctor`.
* [x] Compilation to CommonJS for CLI usage.
* [x] **Agent Locks System (`metadata.json`):** Added a locking mechanism (`lock` and `unlock` commands) to worktrees to prevent multiple agents from stepping on each other, storing `{"lockedBy": "agent-id", "timestamp": "..."}` inside `.worktrees/<feature>/metadata.json`.
* [x] **Garbage Collection Command (`gc`):** Implemented a `wt-agent gc` command to automatically find and remove worktrees for features that have already been merged into `main`, saving disk space.
* [x] **Native Pre/Post Action Hooks:** Added a `--hook` parameter to `sync` and `create` to execute commands after a successful action (e.g., `--hook "npm ci && npm run test"`).
* [x] **Testing:** Added a comprehensive test suite using `vitest` with isolated temporary git repositories to verify the deterministic creation, locking, synchronization, and removal mechanisms.
* [x] **Interactive Mode:** Implemented an interactive CLI mode using `prompts` for human usage, accessible via `wt-agent interactive` or `wt-agent i`.
* [x] **CI/CD Integration:** Configured GitHub Actions (`.github/workflows/ci.yml`) to automatically run tests across different Node versions on every Push or Pull Request.
* [x] **System Checkpoints (Lightweight Snapshots):** Implemented `checkpoint-save <message>` and `checkpoint-restore` commands to capture and discard experimental uncommitted code without polluting the global Git stash.
* [x] **Dependency Sandboxing:** Added an `--isolate-deps` flag to `create` and `sync` commands to automatically perform local `npm install` inside the worktree if a `package.json` is detected.
* [x] **Structured Conflict Resolution API:** Added `wt-agent status <feature>` to parse `git status --porcelain` into a JSON structure categorizing `conflicts`, `modified`, `added`, and `deleted` files. Added `wt-agent resolve-continue <feature>` to resume operations smoothly after the agent edits the conflicted files.
* [x] **Context Metadata & Agent Handoff:** Expanded the `metadata.json` architecture. Agents can leave messages (`wt-agent context-set "Goal: Fix UI"`) and explicitly transfer ownership of a worktree lock to another agent (`wt-agent handoff <feature> <agentId>`).
* [x] **Dynamic Port Leasing:** Implemented `wt-agent port-request <feature>` which safely allocates an exclusive local port (from 8000 to 8050) ensuring parallel agents don't crash when running local dev servers simultaneously. Ports are tracked in metadata and written automatically to a `.env.local` file inside the worktree.

### Pending Tasks / Future Extensions
* [ ] **Plugin System:** Develop an architecture for `wt-agent` plugins so external logic (like interacting with Jira, Linear, or Slack) can be easily integrated into the `sync` or `remove` lifecycle hooks.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pnvdev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pnvdev)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
