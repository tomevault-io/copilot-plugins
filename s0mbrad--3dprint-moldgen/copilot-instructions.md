## react-frontend

> React/TypeScript frontend coding standards for frontend/ files


# React Frontend Standards

- Functional components only, named exports
- Zustand stores: `create<StateType>((set) => ({...}))`, flat structure
- API hooks: `useQuery` for GET, `useMutation` for POST/PUT/DELETE
- Styling: Tailwind utility classes, `cn()` for conditional. Theme vars: `bg-bg-panel`, `text-text-primary`, etc.
- Animation: Framer Motion `<motion.div>`, `<AnimatePresence>`
- Icons: Lucide React — import individual icons
- No `any` type except in legacy code. Use `unknown` + type guards.
- All user-facing text in Chinese

---
> Source: [S0mbraD/3DPrint_MoldGen](https://github.com/S0mbraD/3DPrint_MoldGen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
