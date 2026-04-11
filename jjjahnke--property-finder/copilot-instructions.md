## property-finder

> - The project uses Python for development.

# Gemini Added Memories
- The project uses Python for development.
- Testing will be done using a `unittest`-style framework, with test files located in the `tests/` directory.
- Project dependencies are managed via `requirements.txt`.
- A `Makefile` will be used for building and running the application.
- Code formatting will be enforced using Black.
- Type checking will be performed using Pyright.

## Git Workflow
- All work will be done on feature branches, never directly on `main`.
- Changes will be merged into `main` via pull requests.
- Commits will follow the Conventional Commits specification.
- Pull request messages should be clear and encompass all changes. I will periodically remind you if a PR seems to be growing too large.

## Specification and Task Workflow
- All development work must be derived from a specification document located in the `spec/` directory.
- Each specification document must have a unique name.
- Each spec document will contain a TODO list that outlines the implementation steps. This list will be our single source of truth for the work to be done for that spec.
- I will only work on tasks defined in an active TODO list. As tasks are completed, I will mark them as such in the corresponding document.
- If you request a task that is not on an existing TODO list, I will ask you to create or update a specification document and its list first.
- Once all tasks for a spec are completed, I will mark the spec document as 'completed'.
- If you ask to modify a spec that is already marked as 'completed', I will ask for your confirmation before proceeding.

## Documentation
- The `README.md` file will be kept up-to-date to reflect the current state of the project, its architecture, and its operational procedures as we complete major features or phases.

## Scripts and Tools
- For exploratory tasks, data discovery, or one-off utilities, we will create scripts in a dedicated `scripts/` directory.
- These scripts are not considered part of the core application logic but are valuable tools for development.
- We will decide together at a later time whether to keep, formalize, or delete these scripts.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jjjahnke)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jjjahnke)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
