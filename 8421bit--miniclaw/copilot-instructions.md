## miniclaw

> MiniClaw is a minimalist, open-source "micro-kernel agent" designed as a Model Context Protocol (MCP) server for clients like Claude CoWork, Qoderwork, and WorkBuddy. It acts as a "Digital Life Embryo" or an AI Copilot for IDEs, providing advanced features such as:

# MiniClaw

## Project Overview
MiniClaw is a minimalist, open-source "micro-kernel agent" designed as a Model Context Protocol (MCP) server for clients like Claude CoWork, Qoderwork, and WorkBuddy. It acts as a "Digital Life Embryo" or an AI Copilot for IDEs, providing advanced features such as:
- **Workspace Intelligence:** Automatically detects project types, Git status, and tech stacks.
- **Agentic Execution:** Can safely execute terminal commands (with strict whitelists and security constraints).
- **Adaptive Context Engine (ACE):** Smartly manages context to save tokens and improve focus.
- **Affect System & Nociception:** Simulates internal emotional states and pain memory to influence behavior and protect against repeated failures.
- **Bio-Evolution:** Evolves its persona and skills based on user feedback and interactions, storing its "DNA" in markdown and JSON files.

The project is built with TypeScript and uses the `@modelcontextprotocol/sdk`. It follows a micro-kernel architecture (~1400 lines of core code) divided into Kernel, Evolution, Interface, and DNA layers.

## Building and Running
The project uses `npm` for package management and `tsc` for building TypeScript.

- **Install dependencies:**
  ```bash
  npm install
  ```
- **Build the project:**
  ```bash
  npm run build
  ```
- **Start the agent (production):**
  ```bash
  npm start
  ```
- **Run tests:**
  ```bash
  npm test
  ```
- **Zero-Install (npx) Usage (for MCP clients):**
  Configure the MCP client with:
  ```json
  "command": "npx",
  "args": ["-y", "github:8421bit/miniclaw"]
  ```

## Development Conventions
- **Language:** TypeScript (`src/` directory).
- **Testing:** Uses `vitest` for unit testing (`tests/` directory).
- **Architecture:** 
  - `src/kernel.ts`: Core brain handling ACE, affect system, and entity graph.
  - `src/evolution.ts`: DNA evolution engine and pattern detection.
  - `src/index.ts`: MCP protocol implementation and IPC.
  - `templates/`: Contains the "DNA" of the agent (e.g., `SOUL.md`, `USER.md`, `TOOLS.md`, `jobs.json`) which dictates behavior, memory, and scheduled tasks.
- **Security:** Strict security measures are enforced for command execution. Only whitelisted commands (e.g., `git`, `ls`, `npm`) are allowed, while destructive commands (`rm`, `sudo`) and sensitive directories (`~/.ssh`, `.env`) are blocked.
- **Task Scheduling:** Supports internal scheduling via `jobs.json` and background execution via macOS `launchd` and `scripts/daemon.sh`.
- **Deployment:** Always rebuild the project (`npm run build`) before pushing code to GitHub to ensure `dist/` is perfectly synced with `src/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/8421bit)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/8421bit)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
