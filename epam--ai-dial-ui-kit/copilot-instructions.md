## ui-kit-react

> React/TypeScript UI kit conventions — components, stories, tests, exports


# AI DIAL UI Kit — TS/React

- **TypeScript**: After edits in this repo, run **`npm run typecheck`**. Fix all reported errors before finishing (including in `*.stories.tsx` / `*.spec.tsx` you touched). Prefer strict typings over `any`; match existing `Dial*` prop patterns.
- **Exports**: Public API lives in `src/index.ts`. Export `Dial*` components and `export type { …Props }` when needed.
- **Components**: `src/components/<PascalName>/` with `.tsx` implementation; use `Dial` prefix for exported names.
- **Stories**: `*.stories.tsx` alongside the component; use Storybook 10 patterns already in the repo.
- **Tests**: `*.spec.tsx` or `__tests__/` with Vitest + Testing Library; mirror user-visible behavior and a11y where relevant.
- **Styles**: Tailwind + `tailwind-merge` / `mergeClasses` where the codebase already does; follow `eslint-plugin-tailwindcss` rules.
- **Types**: Shared types under `src/types/`; co-locate small types in the component file if local only.

---
> Source: [epam/ai-dial-ui-kit](https://github.com/epam/ai-dial-ui-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
