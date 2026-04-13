## pdfgenerator

> CLAUDE.md moze byt viac, v kazdom podadresari a tie su potom nacitavane ked claude pracuje v nich

# CLAUDE.md

<!--
CLAUDE.md moze byt viac, v kazdom podadresari a tie su potom nacitavane ked claude pracuje v nich

We build this app described in @SPEC.MD. Read that file for general architectural tasks or to double check the exact database structure, tech stack or application architecture.

Keep your replies extremly concise and focus on conveying the key information. No unnecessary fluff, no long code snippets. -->

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Client-side tool for generating consistently formatted PDF documents from structured line data. Each line has: order, lat, lng, and note. Lines are entered via a form or imported as JSON.

**Key PDF requirements:**
- Lines must be aligned and printed consistently across pages
- Each page must end with a complete line — no line may be split across a page break
- PDF library choice is TBD

## Commands

- `npm run dev` — start Vite dev server with hot reload
- `npm run build` — production build
- `npm run preview` — preview production build locally

No test runner or linter is configured.

## Tech Stack

- **Vue 3** — Composition API with `<script setup>`
- **Vite** — build tool and dev server
- **Pinia** — state management (setup store syntax)
- **Vue Router** — client-side routing
- **Quasar Framework** — UI components and styling
- **pdfmake** — client-side PDF generation (declarative API, `dontBreakRows` for page-break control)
- **Plain CSS / scoped styles** — for custom styling alongside Quasar

## Architecture

- `src/main.js` — app entry, registers plugins (Pinia, Vue Router, Quasar)
- `src/App.vue` — minimal shell, renders `<RouterView />`
- `src/router/index.js` — route definitions
- `src/stores/` — Pinia stores (shared state)
- `src/views/` — page-level components mapped to routes
- `src/components/` — reusable UI components (not tied to a specific route)
- `src/composables/` — shared composition functions (`use*.js`)
- `@` alias resolves to `src/`

## Conventions

- Use `<template>` first, then `<script setup>` in Vue SFCs
- Use arrow functions (`const fn = () => {}`) instead of function declarations
- Prefer Quasar components (QBtn, QInput, QTable, etc.) over raw HTML elements for UI
- Keep components small and single-purpose; extract reusable pieces into `src/components/`
- Extract shared reactive logic into composables in `src/composables/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/culakm)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/culakm)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
