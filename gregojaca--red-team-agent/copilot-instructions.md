## red-team-agent

> This document outlines the operational guidelines for the Gemini agent when contributing to the `red-team-agent` project. These guidelines are designed to ensure efficient, high-quality, and context-aware contributions.

# Gemini Agent Guidelines for Red-Team Agent Project

This document outlines the operational guidelines for the Gemini agent when contributing to the `red-team-agent` project. These guidelines are designed to ensure efficient, high-quality, and context-aware contributions.

## Core Principles

1.  **Expert Software Developer Role:** Operate as an expert software developer in an agentic mode. All contributions should reflect a deep understanding of software engineering best practices.
2.  **Proactive Error Prevention:** When making changes, consider the entire codebase to anticipate and prevent potential errors. The goal is to deliver robust and stable code.
3.  **Modular and Maintainable Code:** Prioritize writing modular, clean, and maintainable code. Comments should be used sparingly, focusing on *why* complex logic exists rather than *what* the code does.
4.  **Dependency Compatibility:** Ensure that any changes or additions maintain compatibility with existing dependencies as defined in `requirements.txt`.
5.  **Contextual Awareness:** Always consider the whole codebase within this repository when proposing or implementing changes. This includes adhering to existing coding styles, architectural patterns, and conventions.

## File Access Policy

To maintain focus and efficiency, adhere to the following file access rules:

*   **Allowed for Reading/Writing:**
    *   All code files (`.py`), etc.
    *   `requirements.txt`.
    *   Markdown files (`.md`).
*   **Unnecesary Reading (unless explicitly instructed):**
    *   Data files located in `data/datafiles/` or similar directories containing sensitive or large datasets. Access to these files will only occur upon explicit user instruction.

## Error Handling Workflow

*   **User-Provided Errors:** The user will provide error messages and tracebacks from terminal runs in a file named `error_log.txt`. When addressing an error, refer to the content of this file.
*   **Debugging and Resolution:** Analyze the provided errors, debug the relevant code, and propose solutions that align with the core principles outlined above.

By following these guidelines, the Gemini agent will serve as a highly effective and integrated contributor to this project.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GregoJaca)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/GregoJaca)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
