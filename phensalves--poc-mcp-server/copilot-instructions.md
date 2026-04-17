## poc-mcp-server

> This document outlines specific guidelines for the Gemini agent when interacting with this project.

# Gemini Agent Guidelines for this Project

This document outlines specific guidelines for the Gemini agent when interacting with this project.

## Commit Message Guidelines

When performing `git commit` operations, adhere to the following principles for well-documented and organized commit history:

1.  **Organize by Context**: Group related file changes into a single logical commit. Avoid monolithic commits that mix unrelated changes.
    *   **Example Contexts**:
        *   `feat`: New features.
        *   `fix`: Bug fixes.
        *   `refactor`: Code refactoring (no new features or bug fixes).
        *   `docs`: Documentation changes (e.g., README, wiki).
        *   `test`: Adding or modifying tests.
        *   `ci`: CI/CD pipeline changes.
        *   `build`: Build system or external dependencies changes.
        *   `chore`: Miscellaneous tasks (e.g., updating `.gitignore`).

2.  **Conventional Commits**: Use the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) specification for commit messages.
    *   **Format**: `<type>(<scope>): <subject>`
        *   `<type>`: `feat`, `fix`, `refactor`, `docs`, `test`, `ci`, `build`, `chore`.
        *   `<scope>` (optional): A short description of the part of the codebase affected (e.g., `frontend`, `api`, `python-analyzer`, `docker`).
        *   `<subject>`: A very short, imperative mood summary of the change.
    *   **Body**: Provide a more detailed explanation of the change, including:
        *   **Why** the change was made.
        *   **What** problem it solves.
        *   **How** it was implemented (briefly).
        *   Any relevant **trade-offs** or **impacts**.

3.  **Detailed File Staging**: Before committing, use `git add` to carefully stage only the files relevant to the current logical change.

## Overriding Guidelines

These guidelines are the default behavior. However, if the user provides custom instructions regarding commit messages or file organization, **always prioritize the user's explicit instructions** over these default guidelines.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/phensalves) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
