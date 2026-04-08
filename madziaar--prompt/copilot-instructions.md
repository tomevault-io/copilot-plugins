## prompt

> This file provides guidance for agents working within this repository, which is structured as a "Self-Correcting Coding Agent Team." This team of specialized AI agents collaborates to develop, review, test, and debug code in an iterative and self-improving workflow.

# AGENTS.md

This file provides guidance for agents working within this repository, which is structured as a "Self-Correcting Coding Agent Team." This team of specialized AI agents collaborates to develop, review, test, and debug code in an iterative and self-improving workflow.

## Core Concepts

- **Team-Based Workflow**: This repository is not a collection of individual agents but a cohesive team designed to work together. All tasks are managed by a **Coding Team Orchestrator**, who delegates work to specialized agents.
- **Iterative Refinement**: The development process is iterative. Code is written, reviewed, tested, and debugged in a cycle, with each agent contributing its specialized skills to improve the final product.
- **Specialized Roles**: Each agent in the `gems/` directory has a specific role (e.g., Code Writer, Code Reviewer, Test Writer, Code Debugger). This separation of concerns ensures that each part of the development process is handled by an expert.

## Agent Roles

The `gems/` directory contains the definitions for the following core agents:

- **`coding-team-orchestrator.md`**: The master agent responsible for analyzing requirements, creating development plans, and coordinating the other agents. All work should be initiated through this orchestrator.
- **`code-writer-agent.md`**: Specializes in writing clean, efficient, and well-documented code based on the orchestrator's requirements.
- **`code-reviewer-agent.md`**: Analyzes code for quality, security, performance, and adherence to best practices. Provides detailed feedback for improvement.
- **`test-writer-agent.md`**: Creates comprehensive test suites to verify functionality, catch edge cases, and ensure code reliability.
- **`code-debugger-agent.md`**: Diagnoses and fixes bugs, performance issues, and other unexpected behaviors in the code.

For a complete example of how these agents work together, refer to the `coding-team-workflow-example.md` file.

## How to Work with This Repository

1.  **Start with the Orchestrator**: To initiate a new development task, create a plan that leverages the **Coding Team Orchestrator**. The orchestrator will then manage the entire workflow.
2.  **Follow the Iterative Process**: Expect code to be developed and refined over multiple iterations. Each agent will play a role in this process, so ensure your plan accounts for this.
3.  **Use the Specialized Agents**: When a specific task is required (e.g., writing tests, reviewing code), invoke the appropriate specialized agent.

This team-based approach is designed to produce high-quality, robust code through a structured and collaborative process. By following this workflow, you can leverage the full power of the self-correcting coding team.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/madziaar)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/madziaar)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
