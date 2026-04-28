## epstein-studio

> Operational instructions for coding agents working in this repository.

# AGENTS.md

## Purpose
Operational instructions for coding agents working in this repository.

## Project Context
- Stack: Django backend with server-rendered templates and static JS/CSS.
- Main app: `backend/apps/epstein_ui`.
- Runtime package manager: `uv`.
- Primary branch: `master`.

## Documentation Contract (Required)
- For every code change, update the relevant documentation file in `docs/`.
- If no existing file fits the change scope, create a new doc in `docs/` and link it from this file.
- Keep docs in sync with actual behavior, commands, and paths.
- Do not defer doc updates to a later commit unless explicitly requested.

## Docs Map
- `docs/dev.md`: local development setup and command workflow.
- `docs/deploy.md`: production deployment and update flow.
- `docs/architecture.md`: system structure and data flow.
- `docs/operations.md`: maintenance and troubleshooting playbooks.

## Working Rules
- Keep changes focused and minimal.
- Preserve existing UI/UX patterns unless explicitly asked to redesign.
- Do not revert unrelated local changes.
- Prefer non-interactive git commands.
- Use `rg` for search.

## Branching and Commits
- Create feature branches for non-trivial work.
- Use clear commit messages describing user-visible intent.
- Do not amend existing commits unless requested.

## Django and Static Assets
- Use `{% static %}` in templates for static references.
- Avoid hardcoded `/static/...` paths when template resolution is possible.
- If new static files are added, ensure deployment docs mention `collectstatic`.

## Safety
- Never use destructive commands (e.g. `git reset --hard`) unless explicitly requested.
- Stop and ask if unexpected repo state changes appear.

---
> Source: [artischocki/epstein-studio](https://github.com/artischocki/epstein-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
