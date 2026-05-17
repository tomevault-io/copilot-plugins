## vue-sfc

> Vue single-file component layout


# Vue SFC layout

- The opening tag must be exactly `<script setup lang="ts">` (not `<script lang="ts" setup>`).
- That `<script setup lang="ts">` block is always the **first** top-level block in the file — it appears above `<template>` and any `<style>` blocks.
- For pages or layouts with no script logic yet, keep an empty `<script setup lang="ts"></script>` so order stays consistent.

---
> Source: [owlsdepartment/flyva](https://github.com/owlsdepartment/flyva) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
