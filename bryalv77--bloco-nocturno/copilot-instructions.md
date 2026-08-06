## bloco-nocturno

> - **ONLY use components from `src/components/ui`.**

# AGENTS.md

# Mandatory Rules

## UI (Highest Priority)

- **ONLY use components from `src/components/ui`.**
- These components are the project's **shadcn/ui** components.
- **Do NOT create custom UI components** if an equivalent exists in `src/components/ui`.
- **Do NOT use any other component library.**
- **Do NOT duplicate existing components.**
- Use **Tailwind CSS only** for styling.

## Code

- Keep code simple, readable and maintainable.
- Follow DRY and SOLID when appropriate.
- Remove unused code, imports and variables.

## TypeScript

- Never use `any`.
- Never use `@ts-ignore`.
- Never disable type checking.

## React

- Use functional components only.
- Keep components small and reusable.
- Extract reusable logic into hooks.

## Quality Gates

Every task must pass before it is considered complete:

```bash
npm run typecheck
```

Never finish a task if either command fails.

---
> Source: [bryalv77/bloco-nocturno](https://github.com/bryalv77/bloco-nocturno) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
