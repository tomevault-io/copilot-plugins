## task-flow

> - Follow: plan → implement → review.

# AGENTS.md

## Workflow

- Follow: plan → implement → review.
- For non-trivial tasks, briefly explain the plan before editing code.
- Keep changes minimal, focused, and local.
- Do not refactor unless explicitly requested or required to complete the task safely.
- Prefer the simplest reliable solution over a more abstract one.

## Source of truth

- Use the project codebase as the primary source of truth.
- Use official docs when behavior depends on a library, framework, API, or version.
- Use RAG if configured and relevant to the task.
- Prefer disabling unsupported mobile features over introducing complex partial implementations.

## Validation

- Run lint after code changes when a lint script exists.
- Do not run a full build unless necessary, requested, or the change affects build/runtime configuration.
- If validation cannot be run, explain why and mention the residual risk.
- For UI changes in shadcn/ui projects, review the result for consistency with existing shadcn/ui components, variants, spacing, and tokens.


## Language

- The user may write in Russian.
- Think and reason internally in English.
- Keep code, identifiers, comments, commits, and technical terms in English.
- Explain decisions in Russian when the user writes in Russian.
- 
## Frontend / UI

- Prefer existing project UI components and patterns before creating new ones.
- When the project uses shadcn/ui, check new or changed UI against shadcn/ui conventions.
- Reuse shadcn/ui components where appropriate instead of recreating them manually.
- Do not add new shadcn/ui components unless they are needed for the task.
- Keep styling consistent with existing Tailwind/shadcn tokens and variants.
- 
## Architecture

- Keep app-level code focused on bootstrap, providers, store setup, routing, and page composition.
- Keep feature business logic inside the owning feature folder.
- Do not place feature-specific helpers, API logic, or domain logic inside `src/app`.
- Shared/common components must not depend on app slices, feature APIs, or feature-specific state.
- If a shared component needs app/feature behavior, pass state and handlers through props or move the component closer to its owner.
- Avoid fragile file and folder names, including quotes, spaces, and special characters.
- Keep root-level `src` files limited to entrypoints and truly global infrastructure.
- Place reusable UI in `common/components` or the configured shadcn/ui location.
- Place feature-specific UI, hooks, types, and helpers inside the owning feature.

## shadcn/ui

- Keep `components.json` aliases synchronized with the actual project structure.
- Before adding or changing UI, check whether the project uses shadcn/ui components.
- Reuse existing shadcn/ui components and variants where appropriate.
- Do not manually recreate components that already exist in shadcn/ui unless there is a clear project-specific reason.
- Keep Tailwind classes consistent with existing shadcn tokens, spacing, variants, and component patterns.

---
> Source: [SemenKr/task-flow](https://github.com/SemenKr/task-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
