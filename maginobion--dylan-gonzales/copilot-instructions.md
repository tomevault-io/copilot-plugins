## dylan-gonzales

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Personal portfolio website for Dylan Gonzales, built with **Nuxt 3** (with Nuxt 4 compatibility enabled), **Vue 3 Composition API**, **TypeScript**, and **UnoCSS**. Uses **pnpm** as the package manager.

## Commands

```bash
pnpm dev          # Dev server at localhost:3000
pnpm build        # Production build
pnpm generate     # Static site generation
pnpm preview      # Preview production build
```

## Architecture

- **`app/pages/`** — Four pages: index, about, contact, projects
- **`app/components/`** — Auto-imported Vue components (PascalCase filenames)
- **`app/layouts/default.vue`** — Single layout wrapping all pages
- **`app/assets/css/styles.css`** — Global styles with CSS custom properties for theming (dark/light mode via `.dark-mode` class)
- **`server/api/mailer.ts`** — Nodemailer endpoint for the contact form (Mailtrap SMTP)
- **`i18n/locales/`** — English (`en-US.json`) and Spanish (`es-ES.json`) translation files, lazy-loaded via `@nuxtjs/i18n`

## Key Patterns

- **Theming:** All colors use `--color-*` CSS variables; dark mode toggles via `.dark-mode` class on root. Theme state managed through `useAppConfig()`.
- **i18n:** Text uses the `v-t` directive or `$t()`. Browser language auto-detected with cookie persistence. Both locales must be kept in sync.
- **Animations:** Scroll-triggered via Intersection Observer API in `techDisplay.vue` and `project-display.vue`.
- **Environment variables:** `MAILTRAP_USERNAME` and `MAILTRAP_PASSWORD` accessed via `useRuntimeConfig()` in server routes.

## Conventions

- Git commits follow conventional commit format (e.g., `bump(deps)`, `fix(docs)`)
- UnoCSS with Attributify mode enabled — utility classes can be used as HTML attributes
- Prettier for formatting, ESLint for import organization

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Maginobion)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Maginobion)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
