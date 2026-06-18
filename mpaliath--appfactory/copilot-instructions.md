## appfactory

> For all code under this repository:

# AGENTS.md

## iOS miniapp organization rules

For all code under this repository:

1. Keep `ContentView.swift` lightweight and focused on app-level composition only.
2. Place all miniapp implementation code under `HelloWorldiOS/HelloWorldiOS/miniapps/{appName}/`.
   - Example: the Todo miniapp must live under `HelloWorldiOS/HelloWorldiOS/miniapps/todo/`.
3. Follow the Single Responsibility Principle:
   - Keep files small and focused.
   - Prefer splitting UI into small `View` types and keeping models in dedicated files.
   - Avoid large, monolithic view files when adding or updating miniapps.

---
> Source: [mpaliath/AppFactory](https://github.com/mpaliath/AppFactory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
