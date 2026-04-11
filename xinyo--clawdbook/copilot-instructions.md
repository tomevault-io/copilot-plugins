## clawdbook

> - When the #plan command is triggered, ask for confirmation before executing the plan.

## Engineering Principles

- When the #plan command is triggered, ask for confirmation before executing the plan.

- Prefer `pnpm` when installing dependencies.
- Prefer use `Vite` for project initialisation.
- Use `pnpm create vite@latest <project_name> -- --template react-ts` to spin up a new React + Vite package with TypeScript checks ready


- Find the CI plan in the .github/workflows folder
- Use `vitest` for the testing framework with `@testing-library/react` for component testing
- All functional changes should be covered by unit tests
- Always run `pnpm test` after modifying JavaScript/Typescript files
- React Component Test files should be placed alongside the component they test with `.test.tsx` extension
- Mock external dependencies like `uuid` in tests


- Every PR must include a changes summary in the description.
- Branch names and PR titles must always follow semantic conventions.
- Ask for confirmation before adding new production dependencies.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xinyo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/xinyo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
