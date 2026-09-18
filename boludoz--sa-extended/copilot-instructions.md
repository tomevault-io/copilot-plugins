## sa-extended

> For pull requests that add or modify reversed classes under `source/game_sa/**`, always review `docs/HookRegistration.md` and enforce its checklist.

# Copilot review instructions

For pull requests that add or modify reversed classes under `source/game_sa/**`, always review `docs/HookRegistration.md` and enforce its checklist.
Make no mistakes!

Required checks for each new reversed class:

1. The class declaration contains `static void InjectHooks();`.
2. `source/InjectHooksMain.cpp` contains a matching `ClassName::InjectHooks();` call inside `InjectHooksMain()`.
3. `friend void InjectHooksMain();` appears only when private hook-wrapper access is required.

---
> Source: [boludoz/sa-extended](https://github.com/boludoz/sa-extended) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
