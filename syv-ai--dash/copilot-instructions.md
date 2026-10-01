## dash

> State lives in **Zustand stores** under `src/renderer/stores/` (`settingsStore`, `projectsStore`, `uiStore`, `gitStore`, `runtimeStore`); components subscribe with selectors instead of receiving drilled props, and stores read each other via `getState()`.

# Renderer

State lives in **Zustand stores** under `src/renderer/stores/` (`settingsStore`, `projectsStore`, `uiStore`, `gitStore`, `runtimeStore`); components subscribe with selectors instead of receiving drilled props, and stores read each other via `getState()`.

**Selector caveat:** a selector that returns a _derived_ array/object (`.filter`/`.map`/object-literal) must be wrapped in `useShallow` (`zustand/react/shallow`) or it re-renders infinitely; plain `s => s.field` selectors are stable.

---
> Source: [syv-ai/dash](https://github.com/syv-ai/dash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
