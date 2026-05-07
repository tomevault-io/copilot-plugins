## frontend-react-typescript-vite-tailwind

> Default frontend stack and implementation standards for this project.

- **Follow `ai_docs/FRONTEND_STANDARDS.md` for full frontend conventions.**
- Fixed stack: **pnpm** + React + TypeScript + Vite + Tailwind CSS + **shadcn/ui**.
- Use shadcn/ui as the component library; customise via Tailwind and CSS variables.
- Build reusable UI primitives in `frontend/src/components/ui`.
- Organize business logic in `frontend/src/features/*`.
- Utilities and API wrappers go in `frontend/src/lib`.
- Use strict typing and avoid `any` except for documented edge cases.
- Prefer semantic HTML, keyboard accessibility, and visible focus states.
- Run `pnpm --prefix frontend lint`, `typecheck`, `test`, and `build` before completion.

---
> Source: [Tendo33/IdeaGo](https://github.com/Tendo33/IdeaGo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
