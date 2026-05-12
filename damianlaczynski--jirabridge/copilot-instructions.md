## jirabridge

> > Scope: fast-start context for AI agents and contributors working in this repository. Load this file first, then open the focused docs under `docs/` for the area you are changing.

# AI Working Guide

> Scope: fast-start context for AI agents and contributors working in this repository. Load this file first, then open the focused docs under `docs/` for the area you are changing.

## Repository shape

- `src/` - source code for the JiraBridge interactive CLI.
- `tests/` - unit and smoke-style fixture tests for the CLI.
- `docs/` - repo-specific implementation, testing, and structure guidance.

## Start here by task

- End-user install and workflows: read `docs/user-guide.md` and root `README.md`.
- CLI or application change: read `docs/repo-map.md` and `docs/backend-coding-guidelines.md`.
- Test work or migration verification: read `docs/testing-playbook.md`.

## Commands

### CLI solution

- Build solution: `dotnet build JiraBridge.sln`
- Run app: `dotnet run --project src/JiraBridge`
- Unit tests: `dotnet test tests/JiraBridge.UnitTests`

Run commands from the repository root.

## Repo navigation rules

### Current template structure

- App entry point lives in `src/JiraBridge/Program.cs`.
- Host and terminal loop live in `src/JiraBridge/Host/`.
- Navigation and command palette live in `src/JiraBridge/Navigation/`.
- Screens and view models live in `src/JiraBridge/Screens/`.
- Use-case orchestration lives in `src/JiraBridge/Application/`.
- Domain models and invariants live in `src/JiraBridge/Domain/`.
- File system, Jira, and sync adapters live in `src/JiraBridge/Infrastructure/`.
- Shared primitives live in `src/JiraBridge/Shared/`.
- Unit tests live in `tests/JiraBridge.UnitTests`.

## Change coupling checklist

- If you change a CLI command shape, check parser, router, help text, and unit tests.
- If you change repository paths or marker names, check domain defaults, infrastructure stores, and docs.
- If you change Jira access, check configuration loading and test coverage.

## Working agreements

- Follow the current folder structure instead of collapsing logic into `Program.cs` or one generic `Services` directory.
- Keep docs current when introducing a new enforced convention.
- Do not assume files visible in the IDE are committed; verify against the filesystem first.

---
> Source: [damianlaczynski/JiraBridge](https://github.com/damianlaczynski/JiraBridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
