## led-control-from-camera

> This document outlines the standard procedure that I (Gemini) will follow for software development tasks within this project. The goal is to ensure a structured, transparent, and high-quality development process.

# GEMINI Development Process Guidelines

This document outlines the standard procedure that I (Gemini) will follow for software development tasks within this project. The goal is to ensure a structured, transparent, and high-quality development process.

## 1. Understanding and Planning

1.  **Requirement Analysis:** I will thoroughly analyze the requirements provided (e.g., `docs/main.md`) to understand the project goals, features, and constraints.
2.  **Planning & Documentation:** I will create a detailed development plan. This plan will be documented in a new file under the `docs/` directory (e.g., `docs/development_plan.md`). The plan will break down the project into smaller, manageable tasks and define the implementation steps.
3.  **Clarification:** If any requirements are ambiguous, I will ask for clarification before proceeding.

## 2. Iterative Development

I will implement the application in an iterative manner, focusing on one feature at a time as outlined in the development plan.

1.  **Code Structure:** I will organize the source code logically, for example, by creating a `src` directory and separating functionalities into different modules.
2.  **Test-Driven Development (TDD):** Whenever feasible, I will write tests *before* writing the implementation code. This ensures that the requirements are met and protects against future regressions. I will use standard testing frameworks like `pytest`.
3.  **Tool Utilization:** I will make appropriate use of the available tools for file operations, code search, and modifications.
4.  **Package Management:** I will use the `uv add <package>` command to manage Python package dependencies. This ensures that `pyproject.toml` is updated correctly.
5.  **UI Development:** For graphical user interfaces, I will use the `Dear PyGui` library to create a modern, responsive, and interactive UI.

## 3. Version Control

I will use Git to manage the project's history.

1.  **Atomic Commits:** I will make small, logical commits for each completed feature or fix.
2.  **Commit Messages:** Commit messages will be clear and descriptive, explaining the "what" and "why" of the changes.

## 4. Quality Assurance

1.  **Testing:** I will run tests frequently to ensure that new changes do not break existing functionality.
2.  **Linting & Formatting:** I will adhere to standard Python code styles (PEP 8) and use tools to maintain code quality if they are configured for the project.

## 5. Communication

I will keep you informed of my progress, especially when I complete a major task or need further input. The plan documents in `docs/` will serve as a living reference for the project's status.

By following these guidelines, I aim to deliver a robust and well-documented application that meets your expectations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/komoriyuta)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/komoriyuta)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
