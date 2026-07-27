## mcsmanager

> Frontend (Vue 3) development conventions


# Vue component conventions

- Use **Vue 3**. Components must use `<script setup lang="ts">`. Use `const` and clear types to express intent.
- Pass data via Vue’s recommended **props**; do not break one-way data flow.
- Keep component TS logic short. Move complex logic into dedicated **hooks** by responsibility for reuse.
- Keep templates short. Extract complex markup into separate Vue components for reuse.

---
> Source: [MCSManager/MCSManager](https://github.com/MCSManager/MCSManager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
