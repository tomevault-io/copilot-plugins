## context

> info on managing state

- global state lives exclusively in `AppContext` (React context + reducer) – do not introduce other global state libraries (redux, recoil, zustand, etc.)
- interact with global state via `useAppContext()` and dispatch predefined action types in `AppContext.tsx`
- for transient ui state (modals, tooltips) use local `useState` in the component

---
> Source: [squarewave-studio/op-patchstudio](https://github.com/squarewave-studio/op-patchstudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
