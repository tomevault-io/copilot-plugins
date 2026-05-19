## agent-james

> This document provides context for agents modifying this codebase regarding the overall architectural direction and operating model of A.D.A.

# Agent Context: A.D.A Architecture and Orchestration

This document provides context for agents modifying this codebase regarding the overall architectural direction and operating model of A.D.A.

## The Goal: "Fleet Commander"
A.D.A is transitioning from a traditional voice-assistant that executes simple scripts (e.g., fetching weather) to an **Autonomous Tech Lead**.

The primary user operates as "The Board", overseeing massive parallel engineering work done by a fleet of 15-75 remote `Jules` agents across multiple repositories. A.D.A's job is to sit in the middle and manage that fleet.

### Key Architectural Concepts
1.  **Interceptor Pattern:** When external agents (like Jules) send updates or ask questions, A.D.A should *not* immediately forward these to the human user. Instead, A.D.A intercepts the message, uses an internal LLM (`OllamaAgent`) to evaluate it against local context (`ProjectManager`), and attempts an auto-reply.
2.  **Escalation Pipeline:** A.D.A only alerts the human user when a Jules task hits a state requiring "Board Approval" (e.g., architectural pivot, missing API keys, or final PR review). This is handled via the `notify_user` system and Slack.
3.  **Proactive Git Review (Pending):** A.D.A should proactively monitor git repositories. When a Jules agent creates a commit or PR, A.D.A should spawn an internal Code Review agent to assess the code and post feedback directly back to Jules, acting as a CI/CD loop.

## Code Constraints
- `backend/ada.py`: The central nervous system. Manages the main Audio Loop, tool registry, and routing logic. Keep heavy logic out of here; delegate to specific agents. The `_handle_jules_triage` method is crucial for the Fleet Commander pattern.
- `backend/jules_agent.py`: The interface to the remote Jules API. Contains polling loops and formatting. It uses callbacks (and specifically `interceptor_callback`) to route messages to A.D.A for triage.
- **Dependency Management:** This project uses specific virtual environments. Ensure you do not arbitrarily add new global dependencies. Testing must be done via the provided `pytest` wrapper script or explicitly activated environment.

## Pull Request Naming
- 🎨 Palette: [UX improvement]
- 🧹 [code health improvement description]
- 🧪 [testing improvement description]

---
> Source: [Cyberdogs7/agent_james](https://github.com/Cyberdogs7/agent_james) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
