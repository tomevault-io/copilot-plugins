## gemini-specs

> This repository uses a Spec-Driven Development (SDD) workflow, managed by a Gemini agent. The goal is to ensure that features are well-defined and planned before implementation.

# Gemini Specs Repository

This repository uses a Spec-Driven Development (SDD) workflow, managed by a Gemini agent. The goal is to ensure that features are well-defined and planned before implementation.

## Spec-Driven Development Workflow

The workflow consists of three main commands that you can issue to the Gemini agent:

1.  **`gemini specify "<feature description>"`**: This is the first step. You provide a high-level description of the feature you want to build. The agent will then create a feature branch and a specification file.

2.  **`gemini plan`**: Once a feature has a specification, this command will generate a detailed implementation plan. This includes defining the technical approach, data models, and contracts.

3.  **`gemini tasks`**: After the plan is complete, this command will break it down into small, executable tasks. These tasks are designed to be clear enough for an LLM to implement.

## Repository Structure

*   `.gemini/commands/`: Contains the definitions for the `specify`, `plan`, and `tasks` commands.
*   `.specify/`: Contains the a core logic and templates for the SDD workflow.
    *   `memory/`: Contains the `constitution.md`, which defines the engineering principles and standards for this project.
    *   `scripts/`: Contains shell scripts that automate parts of the workflow.
    *   `templates/`: Contains templates for the specification, plan, and task files.

## Getting Started

1.  **Define your constitution:** Before you start, fill out the `.specify/memory/constitution.md` file. This document will guide the agent in making design and implementation decisions.
2.  **Specify your first feature:** Run `gemini specify "<your feature description>"` to begin.

## Contribution Guidelines

### Conventional Commits

All commit messages should follow the [Conventional Commits](https://www.conventionalcommits.org/) specification. This helps in automating changelog generation and makes the commit history more readable.

The format is as follows:

```
<type>[optional scope]: <description>

[optional body]

[optional footer]
```

### Branch Naming

Branch names should also follow a conventional format, similar to commit types. This helps in quickly identifying the purpose of a branch.

*   `feat/...`: For new features.
*   `fix/...`: For bug fixes.
*   `docs/...`: For documentation changes.
*   `style/...`: For code style changes.
*   `refactor/...`: For code refactoring.
*   `test/...`: For adding or improving tests.
*   `chore/...`: For routine tasks and maintenance.

## The Constitution

The `.specify/memory/constitution.md` file is the most important file in this repository. It sets the rules and standards that the Gemini agent will follow. Make sure it reflects your project's architectural and quality guidelines.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adslaton)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/adslaton)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
