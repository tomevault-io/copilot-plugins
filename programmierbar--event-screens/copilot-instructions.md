## event-screens

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a [Slidev](https://sli.dev/) presentation project for programmierbar event screens — a looping slide deck displayed at meetup events. Slidev is a Markdown-based slide framework powered by Vue 3 and Vite. The package manager is **npm**.

## Commands

- **Dev server:** `npm dev` (opens slide selection, then a browser, serves at http://localhost:3030)
- **Build:** `npm build` (outputs to `dist/`)
- **Export to PDF:** `npm export`

## Architecture

- `slides.md` — Main slide deck entry point. Slides are separated by `---`. Each slide can have YAML frontmatter for layout/transition config, inline `<style>` blocks, `<script setup>` blocks, and Vue components.
- `theme/` — Custom local theme (`slidev-theme-programmierbar`), referenced as `theme: ./theme` in `slides.md` frontmatter.
  - `theme/layouts/` — Layout Vue components (e.g., `title`, `cover`, `agenda`, `quoted`, `feedback`, `mission`, `transit`). Each layout receives slide content via `<slot />`. Layouts can accept frontmatter fields as props via `defineProps` (e.g., `cover.vue` takes `imageUrl`).
  - `theme/components/` — Theme components auto-imported into slides (e.g., `<Logo />`, `<Autoplay />`, `<TransitSchedule />`).
  - `theme/styles/index.ts` — Style entry point; imports Slidev base CSS and `theme.css`.
  - `theme/styles/theme.css` — Global styles: Montserrat font, background image, heading color (`#cfff00`).
  - `theme/assets/` — Theme-internal assets (background image, logos). Referenced via relative imports from theme components.
- `public/assets/` — Static assets served at `/assets/` (images, SVGs, QR codes). Referenced in slides via `/assets/filename`.

## Key Conventions

- **Scoped styles with slotted content:** Use `:deep()` (not `:slotted()`) to style elements inside slot content, since Slidev wraps markdown in intermediate elements. Use `:slotted()` only when targeting direct top-level slot children.
- **Asset paths:** Use relative paths (`../assets/...`) in theme components. Use `/assets/...` in slides for files in `public/`. The `~` prefix resolves to `@slidev/client/`, not the project root.
- **Autoplay:** Slides use the `<Autoplay :seconds="10" :enabled="true" />` component to auto-advance in a loop.
- **Brand colors:** Primary highlight is `#cfff00` (neon yellow/lime), text is white, font is Montserrat.
- Deployment is configured for Vercel (`vercel.json`) as an SPA.
- `.npmrc` sets `shamefully-hoist=true` and `auto-install-peers=true` (required by Slidev).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/programmierbar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
