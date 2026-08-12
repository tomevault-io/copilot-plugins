## mobie-studio

> Mobie Studio is a lightweight, AI-powered desktop application built with Rust and GPUI. It empowers mobile-first QA and Engineers to run automated mobile tests with zero-effort setup. Instead of writing brittle interaction scripts, users converse with an autonomous agent that navigates the UI to achieve high-level goals.

# Mobie Studio: Agent Vision & Architecture

## Vision
Mobie Studio is a lightweight, AI-powered desktop application built with Rust and GPUI. It empowers mobile-first QA and Engineers to run automated mobile tests with zero-effort setup. Instead of writing brittle interaction scripts, users converse with an autonomous agent that navigates the UI to achieve high-level goals. 

## Key Terms
- **Agent:** The autonomous, LLM-driven entity that interprets user goals, observes the mobile UI, and decides on the next sequence of actions using the `rig-core` framework.
- **UI Dump:** The XML representation of the current screen hierarchy extracted from the device (via Android's `uiautomator`). This serves as the agent's "eyes".
- **Action / Tool:** A specific, atomic interaction sent to the device (e.g., `Tap`, `Swipe`, `Input`, `KeyEvent`, `Observe`) executed as a native tool by the LLM.
- **Implicit Wait & Retry:** The execution loop where the agent acts, observes the resulting state, and autonomously decides whether to retry an action or proceed.
- **Session Memory:** The agent's contextual awareness of recent actions to prevent repetitive failure loops.
- **Exploratory Run:** A conversational session where the user provides a goal and the agent figures out the steps dynamically via multi-step planning (Plan -> Execute).
- **YAML Test Case:** A declarative file generated after a successful exploratory run, documenting the exact steps and assertions for future, repeatable CI/CD execution.

## Architecture & Implementation Standards
The application operates within a single process, utilizing asynchronous Rust tasks to ensure the UI remains responsive during agent operations. Implementation will strictly adhere to the following skill guidelines:

- **Rust Architecture & Quality:** `@systems-programming-rust-project` and `@rust-pro` for robust project structure, modern Rust (1.75+) features, and production-ready systems programming.
- **Concurrency:** `@rust-async-patterns` for handling the asynchronous Agent Engine, Tokio task management, and safe state sharing across threads.
- **Frontend Design:** `@ui-skills` for building a clean, responsive, and opinionated GPUI interface.

### Core Modules
1. **Frontend (GPUI):** Manages the chat interface, LLM provider settings (BYOK model), device selection, and displays the generated YAML outputs.
2. **Agent Engine (Async Task Manager):** Handles the state machine for the active session and multi-step planning, communicating progress back to the UI via asynchronous channels (`mpsc`).
3. **Device Bridge (ADB Interactor):** A dedicated module executing `std::process::Command` calls to local `adb` binaries for device discovery, UI dumping, and action execution.
4. **LLM Client (`rig-core`):** Integrates the `rig-core` AI framework to facilitate native tool calling, replacing the need for manual JSON action parsing. It utilizes `reqwest` to securely interact with LLM providers like OpenRouter, injecting mandatory headers (e.g., `HTTP-Referer`, `X-Title`) required for Bring-Your-Own-Key (BYOK) functionality. This allows the agent to autonomously bind Rust-defined tools directly to the LLM's reasoning process.

## The Agent Loop
Instead of a hardcoded state machine, the Agent Engine utilizes an autonomous, tool-driven loop powered by the `rig-core` framework. When a user submits a high-level goal, the agent initiates a multi-step execution cycle:

1. **Strategic Planning:** The agent decomposes the goal into a sequence of logical sub-goals, which are updated dynamically as the UI state changes.
2. **Autonomous Tool Invocation:** Based on the current sub-goal, the agent decides which atomic tools to invoke (e.g., `Tap`, `Input`, `Swipe`, `Observe`, `KeyEvent`).
3. **Implicit Verification:** After each action, the agent automatically triggers the `Observe` tool to capture the resulting UI state. It then evaluates this new state against its internal objectives.
4. **Session Memory & Loop Prevention:** The agent maintains a persistent `SessionHistory` that tracks recent actions and observations. This prevents repetitive failure loops and allows the agent to self-correct by attempting alternative paths.

The loop is configured with a high iteration limit (e.g., `max_turns(20)`) to allow sufficient time for complex navigation and exploratory tasks. It continues until the goal is achieved, a terminal failure is encountered, or the iteration limit is reached.

## Decision Log
- **Architecture:** Monolithic GPUI + Rust desktop app.
- **AI Framework:** Migrated from manual JSON parsing to `rig-core` for LLM provider abstraction, native tool calling, and structured reasoning.
- **Agent Loop:** Transitioned to an autonomous, tool-driven loop with multi-step planning, session memory tracking, and implicit verification.
- **LLM Strategy:** Bring-Your-Own-Key (BYOK) with OpenRouter support via mandatory custom HTTP headers.
- **Test Output:** Auto-generated declarative YAML test cases after successful exploratory runs.

## Development Workflow
We implement and strictly follow **Git Flow** for managing changes to the project. 
- **New Features & Additions:** Any new feature, enhancement, or change must start with a new branch created from the main development branch (typically `develop`).
  - Example: `git checkout -b feature/my-new-feature develop`
- **Commits:** Follow the Conventional Commits specification for all commit messages.
- **Merging:** Features are developed in isolation and merged back into the development branch via pull requests once complete and validated.

---
> Source: [bayazidsustami/mobie-studio](https://github.com/bayazidsustami/mobie-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
